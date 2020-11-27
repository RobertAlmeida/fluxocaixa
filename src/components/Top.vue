<template>
  <div>
    <div class="container">
      <div class="text-center pt-3"><h2>Fluxo de Caixa</h2></div>

      <div class="row pb-2 pt-2">
        <div class="col-md">
          <label for="dataref">Selecione o mês:</label>
          <select
            class="form-control"
            v-model="mesSelect"
            v-on:change="getLancamentos"
          >
            <option value="01">Janeiro</option>
            <option value="02">Fevereiro</option>
            <option value="03">Março</option>
            <option value="04">Abril</option>
            <option value="05">Maio</option>
            <option value="06">Junho</option>
            <option value="07">Julho</option>
            <option value="08">Agosto</option>
            <option value="09">Setembro</option>
            <option value="10">Outubro</option>
            <option value="11">Novembro</option>
            <option value="12">Dezembro</option>
          </select>
        </div>
        <div class="col-md"></div>
        <div class="col-md"></div>
        <div class="col-md text-right pt-1" id="saldo">
          <label for="saldoatual">Saldo Atual</label>
          <div id="saldoatual" style="font-size: 18px">R$ {{ (entradas - saidas).toFixed(2) }}</div>
        </div>
      </div>
    </div>

    <div class="container">
      <div class="table-responsive">
        <table class="table table-hover table-vcenter">
          <thead class="thead-dark">
            <tr>
              <th scope="col">Data</th>
              <th scope="col">Operação</th>
              <th scope="col">Tipo</th>
              <th scope="col">Valor</th>
              <th scope="col">Motivo</th>
            </tr>
          </thead>
          <tbody id="listLançamento">
            <tr v-for="lanca of lacamentos" :key="lanca.id">
              <td>{{ lanca.date }}</td>
              <td :id="lanca.type">
                {{ lanca.type == "true" ? "Entrada" : "Saída" }}
              </td>
              <td>{{ lanca.pay }}</td>
              <td>R$ {{ (+lanca.value).toFixed(2) }}</td>
              <td>{{ lanca.justify }}</td>
            </tr>
          </tbody>
          <br />
        </table>
      </div>
    </div>

    <!-- Pop UP lançamento caixa  -->

    <b-modal id="modal-1" hide-footer title="Movimentação Caixa">
      <div class="form-group">
        <label for="lancamentoAbertura" class="col-form-label">Data:</label>
        <input
          v-model="movimentacao.date"
          type="date"
          class="form-control"
          id="lancamentoAbertura"
          required
        />
      </div>
      <div class="form-group">
        <label for="tipoMovimentacao" class="col-form-label">Tipo:</label>
        <select
          class="form-control"
          name=""
          v-model="movimentacao.type"
          required
        >
          <option value="true">Entrada</option>
          <option value="false">Saída</option>
        </select>
      </div>
      <div class="form-group">
        <label for="operacaoMovimento" class="col-form-label">Pagamento:</label>
        <select
          class="form-control"
          name=""
          v-model="movimentacao.pay"
          required
        >
          <option value="dinheiro">Dinheiro</option>
          <option value="cartao">Cartão</option>
        </select>
      </div>
      <div class="form-group" id="movimenta">
        <label for="valoresMovimentacao" class="col-form-label">Valor:</label>
        <input
          type="text"
          class="form-control"
          v-model.lazy="movimentacao.value"
          v-money="money"
          required
        />
      </div>
      <div class="form-group" id="movimenta">
        <label for="motivo" class="col-form-label">Motivo:</label>
        <textarea
          class="form-control"
          v-model="movimentacao.justify"
          required
        ></textarea>
      </div>
      <b-button
        class="mt-3"
        variant="outline-primary"
        block
        @click="saveMovimentacao"
        >Lançar</b-button
      >
    </b-modal>

    <div class="container">
      <div class="row justify-content-md-center">
        <div class="col-lg-3 col-sm-6 pt-2">
          <div class="card" id="item-entrada">
            <div class="card-body">
              <div class="media align-items-center">
                <div class="media-body">
                  <h4>ENTRADAS</h4>
                  <span>R$ {{ entradas.toFixed(2) }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="col-lg-3 col-sm-6 pt-2">
          <div class="card" id="item-saida">
            <div class="card-body">
              <div class="media align-items-center">
                <div class="media-body">
                  <h4>SAÍDAS</h4>
                  <span>R$ {{ saidas.toFixed(2) }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="col-lg-3 col-sm-6 pt-2">
          <div class="card" :id="(entradas - saidas) > 0 ? 'item-entrada' : 'item-saida'">
            <div class="card-body">
              <div class="media align-items-center">
                <div class="media-body">
                  <h4>Lucro Bruto</h4>
                  <span>R$ {{ (entradas - saidas).toFixed(2) }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <b-button v-b-modal.modal-1 id="myBtn"
        ><i class="fa fa-plus" data-target="#lancamentosCaixa"></i
      ></b-button>
    </div>
  </div>
</template>

<script>
import 'bootstrap/dist/css/bootstrap.css'
import 'font-awesome/css/font-awesome.css'
import db from '../firebase/index'
import { VMoney } from 'v-money'

export default {
  name: 'Top',
  directives: { money: VMoney },
  data () {
    return {
      saldoAtual: '',
      mesSelect: '',
      movimentacao: {
        date: '',
        day: '',
        mounth: '',
        year: '',
        value: 0,
        type: '',
        pay: '',
        justify: ''
      },
      lacamentos: [],
      saidas: 0,
      entradas: 0,
      money: {
        decimal: '.',
        thousands: '.',
        precision: 2,
        masked: false
      }
    }
  },
  methods: {
    dataFormatada (d) {
      const data = new Date(d)
      const dia = data.getDate()
      const mes = data.getMonth() + 1
      const ano = data.getFullYear()

      return [dia, mes, ano]
    },
    async saveMovimentacao () {
      const move = { ...this.movimentacao }
      const date = move.date.split('-')
      move.day = date[2]
      move.mounth = date[1]
      move.year = date[0]

      if (move.type === '' || move.value === '' || move.date === '') {
        alert('Verifique os campos digitados!')
      } else {
        let end = 'saida'
        if (move.type === 'true') {
          end = 'entrada'
        }
        await db
          .firestore()
          .collection(end)
          .add(move)
          .then(() => {
            this.getLancamentos()
            this.$bvModal.hide('modal-1')
          })
          .catch((error) => {
            console.log(error)
          })
      }
    },
    async getLancamentos () {
      console.log(this.mesSelect)
      this.lacamentos = []
      this.entradas = 0
      this.saidas = 0

      const entrada = await db
        .firestore()
        .collection('entrada')
        .where('mounth', '==', `${this.mesSelect}`)
        .get()
      const saida = await db
        .firestore()
        .collection('saida')
        .where('mounth', '==', `${this.mesSelect}`)
        .get()

      entrada.forEach((e) => {
        const add = e.data()
        add.id = e.id
        this.lacamentos.push(add)
        if (this.entradas === 0) {
          this.entradas = +e.data().value
        } else {
          this.entradas = this.entradas + +e.data().value
        }
      })
      saida.forEach((e) => {
        const add = e.data()
        add.id = e.id
        this.lacamentos.push(add)
        if (this.saidas === 0) {
          this.saidas = +e.data().value
        } else {
          this.saidas = this.saidas + +e.data().value
        }
      })
    }
  },
  mounted () {
    const n = Date.now()
    const date = this.dataFormatada(n)
    this.mesSelect = date[1]
    this.getLancamentos()
  }
}
</script>

<style>
#item-display {
  border-radius: 15px;
  background-color: #fafafa;
}
p {
  display: block;
  height: 2500px;
}

#myBtn {
  display: block;
  position: fixed;
  bottom: 30px;
  right: 20px;
  z-index: 9999;
  border: none;
  outline: none;
  background-color: rgb(45, 124, 241);
  color: white;
  cursor: pointer;
  padding: 15px;
  border-radius: 10px;
}

#myBtn:hover {
  background-color: #555;
}

#true {
  color: green;
}
#false {
  color: red;
}

#item-entrada {
  color: white;
  border-radius: 15px;
  border-color: green;
  background-color: green;

}
#item-saida {
  color: white;
  border-radius: 15px;
  border-color: red;
  background-color: red;

}
#listLançamento {
    max-height: 50px;
}
</style>
