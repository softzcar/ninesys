<template>
  <div>
    <pre style="background-color: pink">{{ dataCalc }}</pre>

    <div>
      <span class="floatme">
        <b-button variant="primary" @click="$bvModal.show(modal)">
          <b-icon icon="credit-card"></b-icon>
        </b-button>
        <!-- {{ this.successMsg }} -->
      </span>
      <span v-if="moneyFormatter(calculated) === '$0.00'" class="floatme">
        <span class="floatme">
          <b-badge variant="success">PAGADO</b-badge>
        </span>
      </span>
      <span v-if="moneyFormatter(calculated) != '$0.00'" class="floatme">
        <strong>Resta:</strong> {{ moneyFormatter(calculated) }}
      </span>
      <span v-if="moneyFormatter(dataCalc.descuento) != '$0.00'" class="floatme">
        <strong>Desc: </strong>{{ moneyFormatter(dataCalc.descuento) }}
      </span>
      <!-- <span
        v-if="
          moneyFormatter(calculated) != '$0.00' &&
          moneyFormatter(dataCalc.descuento) != '$0.00'
        "
      >
        <b-badge variant="success">PAGADO</b-badge>
      </span> -->

      <b-modal :id="modal" :title="title" hide-footer size="lg">
        <b-overlay :show="overlay" spinner-small>
          <b-container>
            <b-row>
              <b-col xs="12" sm="12" md="6" lg="6" xl="6" offset-md="6" offset-lg="6" offset-xl="6">
                <h5>Tasas del día</h5>
                <b-form>
                  <b-form-group label="Peso">
                    <b-form-input type="number" v-model="peso" @change="guardarPeso" />
                  </b-form-group>
                  <b-form-group label="Dólar">
                    <b-form-input type="number" class="mb-4" v-model="dolar" @change="guardarDolar" />
                  </b-form-group>
                </b-form>
              </b-col>
            </b-row>
          </b-container>

          <b-container v-if="tasasCargadas">
            <b-row>
              <b-col class="mb-4">
                <p class="mt-4 mb-4">
                  <b-alert :show="showError" variant="danger">{{
                    errorMsg
                  }}</b-alert>
                  <b-alert :show="showSuccess" class="text-center" variant="success">{{ successMsg }}</b-alert>
                </p>
                <pre style="background-color: chocolate">
                  {{ dataTable.items }}
                </pre>
                <h4 class="mt-4">Histórico</h4>
                <b-table ref="table" responsive small :items="dataTable.items" :fields="dataTable.fields" class="mb-4">
                  <template #cell(moment)="data">
                    {{ formatTimestamp(data.item.moment) }}
                  </template>

                  <template #cell(abono)="data">
                    {{ moneyFormatter(data.item.abono) }}
                  </template>

                  <template #cell(descuento)="data">
                    {{ moneyFormatter(data.item.descuento) }}
                  </template>
                </b-table>

                <h4 class="mt-4">Estado actual</h4>
                <b-list-group>
                  <b-list-group-item>
                    <strong>Total orden: </strong>{{ moneyFormatter(dataCalc.total) }}
                  </b-list-group-item>
                  <b-list-group-item>
                    <strong>Abonado: </strong>{{ moneyFormatter(dataCalc.abono) }}
                  </b-list-group-item>
                  <b-list-group-item>
                    <strong>Descuentos: </strong>{{ moneyFormatter(dataCalc.descuento) }}
                  </b-list-group-item>
                  <b-list-group-item>
                    <strong>Total: </strong>{{ moneyFormatter(dataCalc.total_abono_descuento) }}
                  </b-list-group-item>
                  <b-list-group-item>
                    <strong>Resta:</strong> {{ moneyFormatter(calculated) }}
                  </b-list-group-item>
                </b-list-group>
              </b-col>
            </b-row>

            <b-row>
              <b-col xl="3" lg="3" md="6" sm="12">
                <b-row>
                  <b-col>
                    <hr />
                    <h4>Dólares {{ totalDolares }}</h4>
                  </b-col>
                </b-row>

                <b-row align-h="start">
                  <b-col>
                    <b-form-group id="input-group-1" label="EFECTIVO" label-for="input-dolares-efectivo" class="pl-2">
                      <b-form-input id="input-dolares-efectivo" type="number" step="0.10" min="0"
                        @change="updateMontoAbono" v-model="form.montoDolaresEfectivo"></b-form-input>
                    </b-form-group>
                  </b-col>
                </b-row>
                <b-row align-h="start">
                  <b-col>
                    <b-form-group id="input-group-2" label="ZELLE" label-for="input-dolares-zelle" class="pl-2">
                      <b-form-input id="input-dolares-zelle" type="number" step="0.10" min="0"
                        @change="updateMontoAbono" v-model="form.montoDolaresZelle"></b-form-input>
                    </b-form-group>
                    <b-form-group label="Detalle Zelle">
                      <b-form-input v-model="form.detalleZelle" placeholder="Detalle Zelle"></b-form-input>
                    </b-form-group>
                  </b-col>
                </b-row>

                <b-row align-h="start">
                  <b-col>
                    <b-form-group id="input-group-3" label="BANESCO PANAMA" label-for="input-dolares-zelle"
                      class="pl-2">
                      <b-form-input id="input-dolares-zelle" type="number" step="0.10" min="0"
                        @change="updateMontoAbono" v-model="form.montoDolaresPanama"></b-form-input>
                    </b-form-group>
                    <b-form-group label="Detalle Banesco">
                      <b-form-input v-model="form.detallePanama" placeholder="Detalle Banesco"></b-form-input>
                    </b-form-group>
                  </b-col>
                </b-row>
              </b-col>

              <b-col xl="3" lg="3" md="6" sm="12">
                <b-row>
                  <b-col>
                    <hr />
                    <h4>Pesos {{ totalPesos }}</h4>
                  </b-col>
                </b-row>

                <b-row align-h="start">
                  <b-col>
                    <b-form-group id="input-group-4" label="EFECTIVO" label-for="input-dolares-efectivo" class="pl-2">
                      <b-form-input id="input-pesos-efectivo" type="number" step="0.10" min="0"
                        v-model="form.montoPesosEfectivo" @change="updateMontoAbono"></b-form-input>
                    </b-form-group>
                  </b-col>
                </b-row>
                <b-row align-h="start">
                  <b-col>
                    <b-form-group id="input-group-5" label="TRANSFERENCIA" label-for="input-dolares-zelle" class="pl-2">
                      <b-form-input id="input-pesos-dolares-zelle" type="number" step="0.10" min="0"
                        v-model="form.montoPesosTransferencia" @change="updateMontoAbono"></b-form-input>
                    </b-form-group>
                    <b-form-group label="Detalle Pesos">
                      <b-form-input v-model="form.detallePesosTransferencia" placeholder="Detalle Pesos"></b-form-input>
                    </b-form-group>
                  </b-col>
                </b-row>
              </b-col>

              <b-col xl="3" lg="3" md="6" sm="12" xs="12">
                <b-row>
                  <b-col>
                    <hr />
                    <h4>Bolívares {{ totalBolivares }}</h4>
                  </b-col>
                </b-row>

                <b-row align-h="start">
                  <b-col>
                    <b-form-group id="input-group-6" label="PAGO MOVIL" label-for="input-bolivares-pagomovil "
                      class="pl-2">
                      <b-form-input id="input-bolivares-pagomovil" type="number" step="0.10" min="0"
                        v-model="form.montoBolivaresPagomovil" @change="updateMontoAbono"></b-form-input>
                    </b-form-group>
                    <b-form-group label="Detalle Pagomovil">
                      <b-form-input v-model="form.detallePagomovil" placeholder="Detalle Pagomovil"></b-form-input>
                    </b-form-group>
                  </b-col>
                </b-row>

                <b-row align-h="start">
                  <b-col>
                    <b-form-group id="input-group-6" label="PUNTO" label-for="input-bolivares-punto " class="pl-2">
                      <b-form-input id="input-bolivares-punto" type="number" step="0.10" min="0"
                        v-model="form.montoBolivaresPunto" @change="updateMontoAbono"></b-form-input>
                    </b-form-group>
                  </b-col>
                </b-row>

                <b-row align-h="start">
                  <b-col>
                    <b-form-group id="input-group-6" label="EFECTIVO" label-for="input-bolivares-efectivo "
                      class="pl-2">
                      <b-form-input id="input-bolivares-efectivo" type="number" step="0.10" min="0"
                        v-model="form.montoBolivaresEfectivo" @change="updateMontoAbono"></b-form-input>
                    </b-form-group>
                  </b-col>
                </b-row>

                <b-row align-h="start">
                  <b-col>
                    <b-form-group id="input-group-6" label="TRANSFERENCIA" label-for="input-bolivares-transferencia "
                      class="pl-2">
                      <b-form-input id="input-bolivares-transferencia" type="number" step="0.10" min="0"
                        v-model="form.montoBolivaresTransferencia" @change="updateMontoAbono"></b-form-input>
                    </b-form-group>
                    <b-form-group label="Detalle Transferencia">
                      <b-form-input v-model="form.detalleBolivaresTransferencia"
                        placeholder="Detalle Transferencia"></b-form-input>
                    </b-form-group>
                  </b-col>
                </b-row>
              </b-col>

              <b-col xl="3" lg="3" md="6" sm="12" class="mb-4">
                <b-row>
                  <b-col>
                    <div v-if="
                      this.$store.state.login.dataUser.departamento ===
                      'Administración'
                    ">
                      <hr />
                      <h4>Descuento</h4>
                    </div>
                  </b-col>
                </b-row>
                <b-row align-h="start">
                  <!-- <b-form-input min="0" :disabled="inputDisabled" v-model="value" type="number"
                  placeholder="Abono"></b-form-input>-->
                  <div v-if="
                    this.$store.state.login.dataUser.departamento ===
                    'Administración'
                  ">
                    <b-form-input min="0" lab :disabled="inputDisabled" v-model="valueDescuento" type="number"
                      placeholder="Descuento" class="mt-4 mb-4"></b-form-input>
                  </div>
                  <b-button :disabled="inputDisabled" class="mt-4" variant="success" block
                    @click="hacerAbono">Abonar</b-button>
                </b-row>
              </b-col>
            </b-row>
            <!-- <pre>
             {{ $data.form }}
            </pre> -->
          </b-container>
        </b-overlay>
      </b-modal>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import mixin from '~/mixins/mixins.js'
import { timeout } from 'q'
import { tmpdir } from 'os'

export default {
  mixins: [mixin],

  data() {
    return {
      dolar: this.$store.state.comerce.dolar,
      peso: this.$store.state.comerce.peso,
      inputDisabled: false,
      showSuccess: false,
      showError: false,
      errorMsg: `ERROR`,
      successMsg: 'OK',
      dataTable: [],
      dataCalc: {
        id_orden: this.idorden,
        abono: 0,
        descuento: 0,
        total: 0,
        moment: null,
      },
      valueDescuento: 0,
      size: 'md',
      title: `Abono a la Orden ${this.idorden}`,
      overlay: false,
      form: {
        abono: 0,
        montoDolaresEfectivo: 0,
        montoDolaresZelle: 0,
        detalleZelle: '',
        montoDolaresPanama: 0,
        detallePanama: '',
        montoPesosEfectivo: 0,
        montoPesosTransferencia: 0,
        detallePesosTransferencia: '',
        montoBolivaresEfectivo: 0,
        montoBolivaresPunto: 0,
        montoBolivaresPagomovil: 0,
        detallePagomovil: '',
        montoBolivaresTransferencia: 0,
        detalleBolivaresTransferencia: '',
      },
    }
  },

  computed: {
    tasasCargadas() {
      let cargadas = false
      if (this.dolar > 0 && this.peso > 0) {
        cargadas = true
      }
      return cargadas
    },

    totalDolares() {
      let totalDolares = 0
      let dolaresEfectivo = parseFloat(this.form.montoDolaresEfectivo)
      let dolaresZelle = parseFloat(this.form.montoDolaresZelle)
      let dolaresPanama = parseFloat(this.form.montoDolaresPanama)

      if (!dolaresEfectivo) {
        dolaresEfectivo = 0.0
      }
      if (!dolaresPanama) {
        dolaresPanama = 0.0
      }
      if (!dolaresZelle) {
        dolaresZelle = 0.0
      }

      totalDolares = dolaresEfectivo + dolaresPanama + dolaresZelle
      this.updateMontoAbono()
      return totalDolares.toFixed(2)
    },

    totalPesos() {
      let totalPesos = 0
      let pesosEfectivo = parseFloat(this.form.montoPesosEfectivo)
      let pesosTransferencia = parseFloat(this.form.montoPesosTransferencia)

      if (!pesosEfectivo) {
        pesosEfectivo = 0.0
      }
      if (!pesosTransferencia) {
        pesosTransferencia = 0.0
      }

      totalPesos = pesosEfectivo + pesosTransferencia
      return totalPesos.toFixed(2)
    },

    totalBolivares() {
      let totalBolivares = 0
      let bolivaresEfectivo = parseFloat(this.form.montoBolivaresEfectivo)
      let bolivaresPagomovil = parseFloat(this.form.montoBolivaresPagomovil)
      let bolivaresPunto = parseFloat(this.form.montoBolivaresPunto)
      let bolivaresTransferencia = parseFloat(
        this.form.montoBolivaresTransferencia
      )

      if (!bolivaresEfectivo) {
        bolivaresEfectivo = 0.0
      }

      if (!bolivaresPagomovil) {
        bolivaresPagomovil = 0.0
      }

      if (!bolivaresPunto) {
        bolivaresPunto = 0.0
      }

      if (!bolivaresTransferencia) {
        bolivaresTransferencia = 0.0
      }

      totalBolivares =
        bolivaresEfectivo +
        bolivaresPagomovil +
        bolivaresTransferencia +
        bolivaresPunto
      return totalBolivares.toFixed(2)
    },

    totalAbono() {
      // CALCULO DOLARES
      const montoDolares =
        parseFloat(this.form.montoDolaresEfectivo) +
        parseFloat(this.form.montoDolaresPanama) +
        parseFloat(this.form.montoDolaresZelle)

      // CALCULO EN PESOS
      const montoPesos =
        (parseFloat(this.form.montoPesosEfectivo) +
          parseFloat(this.form.montoPesosTransferencia)) /
        parseFloat(this.peso)

      // CALCULO EN BOLIVARES
      const montoBolivares =
        (parseFloat(this.form.montoBolivaresEfectivo) +
          parseFloat(this.form.montoBolivaresPagomovil) +
          parseFloat(this.form.montoBolivaresPunto) +
          parseFloat(this.form.montoBolivaresTransferencia)) /
        parseFloat(this.dolar)

      let total = montoDolares + montoPesos + montoBolivares

      if (isNaN(total)) total = 0
      return total.toFixed(2)
    },

    calculated() {
      let total = this.floatMe(this.dataCalc.total)
      let abonado = this.floatMe(this.dataCalc.abono)
      let descuento = this.floatMe(this.dataCalc.descuento)
      let nuevoAbono = 0
      let nuevoDescuento = 0

      if (!this.form.abono) {
        nuevoAbono = 0
      } else {
        nuevoAbono = this.floatMe(this.form.abono)
      }

      // if (this.valueDescuento.trim() === '') {
      if (!this.valueDescuento) {
        nuevoDescuento = 0
      } else {
        nuevoDescuento = this.floatMe(this.valueDescuento)
      }

      let resultado = total - abonado - descuento - nuevoAbono - nuevoDescuento

      if (isNaN(resultado)) {
        resultado = 0
      }
      return resultado
    },

    modal: function () {
      const rand = Math.random().toString(36).substring(2, 7)

      return `modal-${rand}`
    },
  },
  methods: {
    guardarPeso(val) {
      // this.peso = val
      this.$store.commit('comerce/setPeso', val)
      // Realiza alguna otra acción, como enviar los datos al servidor
    },

    guardarDolar(val) {
      // this.dolar = val
      this.$store.commit('comerce/setDolar', val)
      // Realiza alguna otra acción, como enviar los datos al servidor
    },

    updateMontoAbono() {
      let newVal
      let montoBolivares
      let montoDolares
      let montoPesos

      // LIMPIAR VALORES ERRONEOS
      if (!this.form.montoBolivaresEfectivo)
        this.form.montoBolivaresEfectivo = 0
      if (!this.form.montoBolivaresPagomovil)
        this.form.montoBolivaresPagomovil = 0
      if (!this.form.montoBolivaresPunto) this.form.montoBolivaresPunto = 0
      if (!this.form.montoBolivaresTransferencia)
        this.form.montoBolivaresTransferencia = 0
      if (!this.form.montoDolaresEfectivo) this.form.montoDolaresEfectivo = 0
      if (!this.form.montoDolaresPanama) this.form.montoDolaresPanama = 0
      if (!this.form.montoDolaresZelle) this.form.montoDolaresZelle = 0
      if (!this.form.montoPesosEfectivo) this.form.montoPesosEfectivo = 0
      if (!this.form.montoPesosTransferencia)
        this.form.montoPesosTransferencia = 0

      // RESET MONTO ABONO
      this.form.abono = 0

      // CALCULO DOLARES
      montoDolares =
        parseFloat(this.form.montoDolaresEfectivo) +
        parseFloat(this.form.montoDolaresPanama) +
        parseFloat(this.form.montoDolaresZelle)

      // CALCULO EN PESOS
      montoPesos =
        (parseFloat(this.form.montoPesosEfectivo) +
          parseFloat(this.form.montoPesosTransferencia)) /
        parseFloat(this.peso)

      // CALCULO EN BOLIVARES
      montoBolivares =
        (parseFloat(this.form.montoBolivaresEfectivo) +
          parseFloat(this.form.montoBolivaresPagomovil) +
          parseFloat(this.form.montoBolivaresPunto) +
          parseFloat(this.form.montoBolivaresTransferencia)) /
        parseFloat(this.dolar)

      // SUMATOORIA DE TODAS LAS MONEDAS
      console.log('dolares', montoDolares)
      console.log('pesos', montoPesos)
      console.log('bolivares', montoBolivares)
      newVal = (montoDolares + montoPesos + montoBolivares).toFixed(2)
      this.form.abono = newVal
      console.log('this.form.abono = ', newVal)
      return newVal
    },

    hacerAbono() {
      let tmpAbono = parseFloat(this.form.abono) // Antiguamente se enviava un solo vaor ahora tenen=mos que envuar todo por semparado...
      let tmpDescuento = parseFloat(this.valueDescuento)

      // if (isNaN(tmpAbono) || this.form.abono.trim() === '' || !this.form.abono) {
      if (isNaN(tmpAbono) || !this.form.abono) {
        tmpAbono = 0
      }

      if (isNaN(tmpDescuento) || !this.valueDescuento) {
        tmpDescuento = 0
      }

      if (!tmpAbono && !tmpDescuento) {
        this.$fire({
          title: 'Error en el monto',
          html: '<p>Ingrese el monto del abono y/o el descuento</p>',
          type: 'warning',
        })
      } else if (this.calculated < 0) {
        this.$fire({
          title: 'Error en el monto',
          html: '<p>El monto ingresado excede el total de la orden</p>',
          type: 'warning',
        })
      } else {
        this.$confirm(
          `Verifique los datos: abono: ${tmpAbono}, descuento: ${tmpDescuento}`,
          'Abono',
          'info'
        ).then(() => {
          this.abonar().then(() => {
            // this.form.abono = ''
            this.valueDescuento = 0
            this.getDataAbonos()
          })
        })
      }
    },

    async abonar() {
      this.showError = false
      this.showSuccess = false

      this.overlay = true
      this.inputDisabled = true
      const data = new URLSearchParams()
      data.set('descuento', this.valueDescuento)
      data.set('id', this.idorden)
      data.set('empleado', this.$store.state.login.dataUser.id_empleado)
      data.set('responsable', this.$store.state.login.dataUser.id_empleado)
      //TODO aqui vamos a enviar cada metodod e pago por separado...
      data.set('tasa_dolar', this.dolar)
      data.set('tasa_peso', this.peso)
      data.set('montoDolaresEfectivo', this.form.montoDolaresEfectivo)
      data.set('montoDolaresZelle', this.form.montoDolaresZelle)
      data.set('montoDolaresPanama', this.form.montoDolaresPanama)
      data.set('montoPesosEfectivo', this.form.montoPesosEfectivo)
      data.set('montoPesosTransferencia', this.form.montoPesosTransferencia)
      data.set('montoBolivaresEfectivo', this.form.montoBolivaresEfectivo)
      data.set('montoBolivaresPunto', this.form.montoBolivaresPunto)
      data.set('montoBolivaresPagomovil', this.form.montoBolivaresPagomovil)
      data.set(
        'montoBolivaresTransferencia',
        this.form.montoBolivaresTransferencia
      )
      data.set('abono', this.form.abono)
      data.set('tipoAbono', 'Abono a orden')

      data.set('detalleZelle', this.form.detalleZelle)
      data.set('detallePanama', this.form.detallePanama)
      data.set('detallePesosTransferencia', this.form.detallePesosTransferencia)
      data.set('detallePagomovil', this.form.detallePagomovil)
      data.set('detalleBolivaresTransferencia', this.form.detalleBolivaresTransferencia)

      console.log(`Data abono: ${data}`)

      await axios
        .post(`${this.$config.API}/orden/abono`, data)
        .then((res) => {
          this.getData().then((res) => {
            this.overlay = false
            this.valueDescuento = 0
            this.form = {
              abono: 0,
              montoDolaresEfectivo: 0,
              montoDolaresZelle: 0,
              montoDolaresPanama: 0,
              montoPesosEfectivo: 0,
              montoPesosTransferencia: 0,
              montoBolivaresEfectivo: 0,
              montoBolivaresPunto: 0,
              montoBolivaresPagomovil: 0,
              montoBolivaresTransferencia: 0,
            }

            if (this.calculated === 0) {
              this.showError = false
              this.inputDisabled = true
              this.successMsg = 'La orden no tiene deuda pendiente'
              this.showSuccess = true
            } else {
              this.showError = false
              this.inputDisabled = false
              this.successMsg = 'El monto ha sido abonado'
              this.showSuccess = true
            }

            this.overlay = false
          })
          this.getData()
        })
        .catch((error) => {
          this.showSuccess = false
          this.showError = true
          this.inputDisabled = false

          if (error.response) {
            // The request was made and the server responded with a status code
            // that falls out of the range of 2xx
            this.successMsg =
              'No se hizo el abono, el servidor no respondió: ' +
              error.response.data
            console.log(error.response.data)
            console.log(error.response.status)
            console.log(error.response.headers)
          } else if (error.request) {
            this.successMsg =
              'No se hizo el abono, el servidor no respondió: ' + error.request
            // The request was made but no response was received
            // `error.request` is an instance of XMLHttpRequest in the browser and an instance of
            // http.ClientRequest in node.js
            console.log(error.request)
          } else {
            // Something happened in setting up the request that triggered an Error
            this.successMsg =
              'No se hizo el abono, el servidor no respondió: ' + error.message
          }
        })
    },

    async enviarAbono() {
      this.overlay = true
      const data = new URLSearchParams()
      data.set('responsable', this.$store.state.login.dataUser.id_empleado)
      // data.set('form', this.form)
      data.set('tasa_dolar', this.dolar)
      data.set('tasa_peso', this.peso)
      data.set('montoDolaresEfectivo', this.form.montoDolaresEfectivo)
      data.set('montoDolaresZelle', this.form.montoDolaresZelle)
      data.set('montoDolaresPanama', this.form.montoDolaresPanama)
      data.set('montoPesosEfectivo', this.form.montoPesosEfectivo)
      data.set('montoPesosTransferencia', this.form.montoPesosTransferencia)
      data.set('montoBolivaresEfectivo', this.form.montoBolivaresEfectivo)
      data.set('montoBolivaresPunto', this.form.montoBolivaresPunto)
      data.set('montoBolivaresPagomovil', this.form.montoBolivaresPagomovil)
      data.set(
        'montoBolivaresTransferencia',
        this.form.montoBolivaresTransferencia
      )
      data.set('abono', this.form.abono)
      data.set('tipoAbono', 'Abono a orden')

      console.log('data:', data)

      await axios.post(`${this.$config.API}/otro-abono`, data).then((res) => {
        this.valueDescuento = 0
        this.form = {
          abono: 0,
          montoDolaresEfectivo: 0,
          montoDolaresZelle: 0,
          montoDolaresPanama: 0,
          montoPesosEfectivo: 0,
          montoPesosTransferencia: 0,
          montoBolivaresEfectivo: 0,
          montoBolivaresPunto: 0,
          montoBolivaresPagomovil: 0,
          montoBolivaresTransferencia: 0,
        }
        this.overlay = false
        // this.getDataReport(this.fechaActual()).then(() => {
        // })
      })
    },

    async getDataAbonos() {
      this.overlay = true
      await axios
        .get(`${this.$config.API}/ordenes/abono-detale/${this.idorden}`)
        .then((resp) => {
          this.dataTable = resp.data
          this.overlay = false
        })
    },

    async getData() {
      this.overlay = true
      await axios
        .get(`${this.$config.API}/ordenes/abono/${this.idorden}`)
        .then((resp) => {
          this.dataCalc = resp.data.data
          /* console.log(
            `Datos cargados respuesta para la orden ${this.idorden}`,
            this.dataCalc
          ) */
          if (this.calculated === 0) {
            this.showError = false
            this.inputDisabled = true
            this.successMsg = 'La orden no tiene deuda pendiente'
            this.showSuccess = true
          }
          this.overlay = false
        })
    },
  },

  mounted() {
    /* this.$root.$on('bv::modal::show', (bvEvent, modalId) => {
      this.getData().then(() => {
        // console.log(`Calculated order: ${this.idorden}`, this.calculated)
      })

      this.getDataAbonos()
    }) */
    this.getData()
    this.getDataAbonos()
    this.$root.$on('bv::modal::show', (bvEvent, modalId) => { })
  },
  props: ['idorden'],
}
</script>
