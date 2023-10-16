<template>
  <div id="app">
    <div class="container mt-5">
      <div class="card">
        <div class="card-body">
          <div class="row align-items-start">
            <div class="col">
              <input class="form-control" placeholder="Цена" v-model="info.price">
            </div>
            <div class="col">
              <input class="form-control" placeholder="Количество" v-model="info.qty">
            </div>
            <div class="col">
              <input class="form-control" placeholder="Сумма" v-model="info.amount">
            </div>
            <div class="col">
              <button class="btn btn-success w-100" @click="send">Отправить</button>
            </div>
          </div>
          <div class="row align-items-start mt-3">
            <div class="col">
              Цена: {{ info.price }}
            </div>
            <div class="col">
              Количество: {{ info.qty }}
            </div>
            <div class="col">
              Сумма: {{ info.amount }}
            </div>
            <div class="col">
              Текущее состояние: <button class="btn btn-info" data-bs-toggle="modal" data-bs-target="#datainfo">Показать</button>
            </div>
          </div>
          <div class="card mt-3 app__logs">
            <div class="card-body">
              <div v-for="(log, index) in logs" :key="index" class="alert alert-primary">{{ log }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>


    <!-- Modal data -->
    <div class="modal fade" id="datainfo">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-body">
            {{ info }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import _ from 'lodash';

export default {
  data() {
    return {
      info: {
        price: null,
        qty: null,
        amount: null,
      },
      nonce: null,
      logs: []
    }
  },
  computed: {
    getInfo() {
      return Object.assign({}, this.info);
    }
  },
  watch: {
    getInfo: {
      handler(newValue, oldValue) {
        for(const key in oldValue) {
          // Если новое значение не равно старому то вызываем обработчик по ключу // handler_price()
          if (oldValue[key] !== newValue[key]) {
            if (this['handler_' + key]) {
              this['handler_' + key](newValue[key], this.info, () => {
                this.logs.unshift(`Изменение поле ${key}. newValue: ${newValue[key]}, oldValue: ${oldValue[key]}`);
              });
            }

            return;
          }
        }
      },
      deep: true
    }
  },
  methods: {
    'handler_price': _.debounce((value, info, callback) => {
      if (info.qty === null) {
        info.qty = 1;
      }

      if (value > 0 && info.qty > 0) {
        info.amount = value * info.qty;

        callback();
      }
    }, 300),
    'handler_qty': _.debounce((value, info, callback) => {
      if (info.price === null) {
        info.price = 1;
      }

      if (value > 0 && info.price > 0) {
        info.amount = value * info.price;

        callback();
      }
    }, 300),
    send() {
      if (this.info.amount % 2 !== 0) {
        alert('Error: число нечетное');

        this.logs.unshift(`Error: число нечетное amount: ${this.info.amount}`);
        
        return;
      }

      setTimeout(() => {
        const oldValueStorage = localStorage['ctrlhack'];

        this._saveLS();

        setTimeout(() => {
          this._loadLS();

          alert('Success: Успешно отправлено');

          this.logs.unshift(`Success: Успешно отправлено. oldValue: ${oldValueStorage}. currentValue ${JSON.stringify({
            price: this.info.price,
            qty: this.info.qty,
            amount: this.info.amount,
            nonce: this.nonce,
          })}`);
        }, 1000);
      }, 1000);
    },
    _saveLS() {
      localStorage['ctrlhack'] = JSON.stringify({
        price: this.info.price,
        qty: this.info.qty,
        amount: this.info.amount,
        nonce: this.nonce + 1
      });
    },
    _loadLS() {
      if (localStorage['ctrlhack']) {
        const info = JSON.parse(localStorage['ctrlhack']);

        this.info.price = info.price;
        this.info.qty = info.qty;
        this.info.amount = info.amount;
        this.nonce = info.nonce || 0;
      }
    }
  },
  beforeMount() {
    this._loadLS();
  }
}
</script>

<style>
.app__logs {
  height: 300px !important;
  overflow-y: scroll;
}
</style>
