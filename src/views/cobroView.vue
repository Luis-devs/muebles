<template>
  <div class="cobro">
    <v-card class="ma-3">
      <v-card-title>
        <v-row justify="space-between" justify-sm="start" class="px-6 my-4">
          <v-col md="3" lg="2" sm="12" cols="auto">
            <v-row class="align-center">
              <v-icon size="x-large" icon="mdi mdi-tag-plus"></v-icon>
              <h1 class="px-3">Cobro</h1>
            </v-row>
          </v-col>
          <v-col
            sm="5"
            md="7"
            lg="8"
            cols="auto"
            class="text-sm-start text-md-end"
          >
            <v-btn
              prepend-icon="mdi-plus"
              color="green"
              @click="dialogCliente = true"
              >Agregar cliente</v-btn
            >
          </v-col>
          <v-col sm="4" md="2" lg="2" cols="auto">
            <v-btn
              color="blue"
              prepend-icon="mdi mdi-plus"
              @click="dialogPrestamo = true"
              >Crear venta</v-btn
            >
          </v-col>
        </v-row>
      </v-card-title>
      <v-card-title>
        <v-col md="6" sm="12">
          <v-text-field
            v-model="searchPrestamo"
            append-inner-icon="mdi-magnify"
            label="Buscar"
            variant="outlined"
            hide-details
          >
          </v-text-field>
        </v-col>
      </v-card-title>
      <v-card-text>
        <v-data-table
          :headers="headers"
          :items.sync="cobros"
          :loading="loadingTable"
          class="elevation-1"
          item-value="_id"
          :search.sync="searchPrestamo"
        >
          <!-- eslint-disable-next-line vue/valid-v-slot -->
          <template v-slot:item.mora="{ value }">
            <v-chip :color="value ? 'red' : 'green'">
              {{ value ? "Atrasado" : "No" }}
            </v-chip>
          </template>
          <!-- eslint-disable-next-line vue/valid-v-slot -->

          <template v-slot:item.actions="{ item }">
            <v-row class="ga-2" no-gutters>
              <v-icon
                size="large"
                @click="abonarFunction(Object.assign({}, item))"
              >
                mdi-cash
              </v-icon>
              <v-btn
                v-if="!item.completado"
                @click="dialogActualizarUbicacion(item._id)"
                class="elevation-0"
                text
                icon
                dark
                density="compact"
              >
                <v-icon size="large">mdi mdi-map-marker-plus</v-icon>
                <v-tooltip activator="parent" location="top"
                  >Actualizar ubicación</v-tooltip
                >
              </v-btn>
              <v-btn
                v-if="item.ubicacionMap.lat != null"
                class="elevation-0"
                :href="`https://www.google.com/maps?q=${item.ubicacionMap.lat},${item.ubicacionMap.lng}`"
                target="_blank"
                icon
                dark
                density="compact"
                text
              >
                <v-icon size="large">mdi mdi-home-map-marker</v-icon>
                <v-tooltip activator="parent" location="top"
                  >Ver ubicación</v-tooltip
                >
              </v-btn>
              <v-icon
                size="large"
                @click="verPrestamoFunction(Object.assign({}, item))"
              >
                mdi-eye
              </v-icon>
            </v-row>
          </template>
        </v-data-table>
        <v-data-table
          :headers="headersAbonoHoy"
          :items="abonosHoy"
          class="elevation-1 mt-12"
        >
          <template v-slot:top>
            <h1 class="text-center">Abonos de hoy</h1>
            <v-row style="padding: 30px">
              <v-chip color="green">
                <h3>
                  {{ "Total Recaudado $" + calcularAbonoTotales(abonosHoy) }}
                </h3>
              </v-chip>
            </v-row>
          </template>
          <!-- eslint-disable-next-line vue/valid-v-slot -->

          <template v-slot:item.monto="{ value }">
            <v-chip color="green">
              {{ "$" + value.toLocaleString() }}
            </v-chip>
          </template>
        </v-data-table>
        <v-col lg="12" md="12" sm="12">
          <canvas class="mb-6" id="graficaAbonoRutas"></canvas>
        </v-col>
        <v-data-table
          :headers="headersAbonoHoy"
          :items="abonosFechaEspecifica"
          class="elevation-1 mt-12"
        >
          <template v-slot:top>
            <h1 class="text-center">Historial de abonos</h1>
            <v-row style="padding: 30px">
              <v-col md="6" sm="12">
                <VueDatePicker v-model="fechaAbonoApi"></VueDatePicker>
              </v-col>
              <v-col md="6" sm="12">
                <v-btn
                  color="blue"
                  prepend-icon="mdi mdi-plus"
                  @click="findCobroEspecifico()"
                  >Buscar</v-btn
                >
              </v-col>
              <v-chip color="green">
                <h3>
                  {{
                    "Total Recaudado $" +
                    calcularAbonoTotales(abonosFechaEspecifica)
                  }}
                </h3>
              </v-chip>
            </v-row>
          </template>
          <!-- eslint-disable-next-line vue/valid-v-slot -->
          <template v-slot:item.monto="{ value }">
            <v-chip color="green">
              {{ "$" + value.toLocaleString() }}
            </v-chip>
          </template>
        </v-data-table>
      </v-card-text>
    </v-card>
    <v-dialog v-model="dialogVerCobro" persistent width="700">
      <v-card>
        <v-card-text>
          <v-container>
            <v-form>
              <v-row>
                <v-progress-linear v-model="skill" color="green" height="25">
                  <template v-slot:default="{ value }">
                    <strong>{{ Math.ceil(value) }}%</strong>
                  </template>
                </v-progress-linear>
                <v-col cols="12">
                  <v-text-field
                    label="Documento"
                    type="text"
                    required
                    variant="outlined"
                    v-model="verPrestamo.cliente.documento"
                    disabled
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6">
                  <v-text-field
                    variant="outlined"
                    label="Nombre"
                    disabled
                    required
                    v-model="verPrestamo.cliente.nombres"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6">
                  <v-text-field
                    variant="outlined"
                    label="Apellido"
                    disabled
                    required
                    v-model="verPrestamo.cliente.apellidos"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6">
                  <v-text-field
                    variant="outlined"
                    label="Ruta"
                    disabled
                    required
                    v-model="verPrestamo.cliente.direccion.nombre"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6">
                  <v-text-field
                    variant="outlined"
                    label="Producto"
                    disabled
                    required
                    v-model="verPrestamo.producto"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6">
                  <v-text-field
                    variant="outlined"
                    label="Fecha de Inicio"
                    disabled
                    required
                    v-model="verPrestamo.fecha_inicio"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6">
                  <v-text-field
                    variant="outlined"
                    label="N° Cuotas"
                    disabled
                    required
                    v-model="verPrestamo.cuotas"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6">
                  <v-text-field
                    variant="outlined"
                    label="Cuotas atrasadas"
                    disabled
                    required
                    v-model="verPrestamo.cuotas_atrasadas"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6">
                  <v-text-field
                    variant="outlined"
                    label="Total"
                    disabled
                    required
                    v-model="verPrestamo.total"
                  ></v-text-field>
                </v-col>
                <v-card-text v-if="verPrestamo.cuotas !== 0">
                  <div class="font-weight-bold ms-1 mb-2">
                    Restante : ${{
                      calcularRestante(verPrestamo.abono, verPrestamo.total)
                    }}
                  </div>
                  <div class="font-weight-bold ms-1 mb-2">
                    Abonado : ${{ calcularAbono(verPrestamo.abono) }}
                  </div>
                  <v-timeline density="compact" align="start">
                    <v-timeline-item
                      v-for="monto in verPrestamo.abono"
                      :key="monto"
                      dot-color="green"
                      size="x-small"
                    >
                      <div class="mb-4">
                        <div class="font-weight-normal">
                          <strong>
                            Abono: ${{ monto.monto.toLocaleString() }}
                          </strong>
                        </div>
                        <div>{{ formatDate(monto.fecha) }}</div>
                      </div>
                    </v-timeline-item>
                  </v-timeline>
                </v-card-text>
              </v-row>
            </v-form>
          </v-container>
        </v-card-text>
        <v-card-actions class="justify-end">
          <v-btn
            color="red-darken-1"
            variant="tonal"
            @click="dialogVerCobro = false"
          >
            Cerrar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="dialogAbonar" persistent width="700">
      <v-card>
        <v-card-text>
          <v-container>
            <v-form ref="formAbono">
              <v-row>
                <v-col cols="12">
                  <v-text-field
                    label="Documento"
                    type="text"
                    required
                    variant="outlined"
                    v-model="cedulaTemp"
                    disabled
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-text-field
                    type="number"
                    label="Monto"
                    :placeholder="
                      montoSugerido
                        ? 'El monto sugerido es ' +
                          montoSugerido.toLocaleString()
                        : 'Ingrese un monto'
                    "
                    min="1"
                    variant="outlined"
                    v-model="formAbono.abono"
                    :rules="cantidadRules"
                  ></v-text-field>
                </v-col>
              </v-row>
            </v-form>
          </v-container>
        </v-card-text>
        <v-card-actions class="justify-end">
          <v-btn
            color="red-darken-1"
            variant="tonal"
            @click="dialogAbonar = false"
          >
            Cerrar
          </v-btn>

          <v-btn
            color="green-darken-1"
            variant="tonal"
            :disabled="disableBtn"
            @click="abonar"
          >
            Crear
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="dialogPrestamo" persistent width="700">
      <v-card>
        <v-card-title> Nueva venta </v-card-title>
        <v-card-text>
          <v-form v-model="valid" ref="formPrestamo">
            <v-row>
              <v-col cols="12">
                <v-autocomplete
                  label="Nombre del cliente"
                  no-data-text="Sin clientes disponible para venta"
                  return-object
                  :items="clientes"
                  :item-title="
                    (item) => {
                      return `${item.nombres} ${item.apellidos}`;
                    }
                  "
                  variant="outlined"
                  v-model="form.cliente"
                  :rules="campoRules"
                ></v-autocomplete>
              </v-col>
              <v-col md="6" cols="12">
                <v-autocomplete
                  label="Producto"
                  return-object
                  no-data-text="Sin productos registrados"
                  item-value="producto._id"
                  :items="productos"
                  item-title="producto.nombre"
                  variant="outlined"
                  v-model="form.producto"
                  :rules="campoRules"
                ></v-autocomplete>
              </v-col>
              <v-col md="6" cols="12">
                <v-text-field
                  type="number"
                  label="Cantidad"
                  placeholder="Ingrese cantidad del producto"
                  min="1"
                  variant="outlined"
                  v-model="form.cantidad"
                  :rules="cantidadRules"
                ></v-text-field>
              </v-col>
              <v-col :cols="cols2[0]">
                <v-select
                  label="Forma de pago"
                  :items="formasPago"
                  item-value="index"
                  item-title="forma"
                  placeholder="Escoja forma de pago"
                  variant="outlined"
                  v-model="formaPago"
                  :rules="campoRules"
                ></v-select>
              </v-col>
              <v-col :cols="cols2[1]" v-if="formaPago == 2">
                <v-text-field
                  type="number"
                  label="Cuotas"
                  placeholder="Ingrese cantidad de cuotas"
                  min="1"
                  variant="outlined"
                  v-model="form.cuotas"
                  :rules="cantidadRules"
                ></v-text-field>
              </v-col>
              <v-col cols="12" v-if="formaPago == 2">
                <v-chip color="green">
                  Fechas seleccionadas:
                  {{ form.pago_fechas.length }}</v-chip
                >
              </v-col>
              <v-col cols="12" v-if="formaPago == 2">
                <VueDatePicker
                  format="yyyy-MM-dd"
                  :rules="campoRules"
                  :enable-time-picker="false"
                  cancelText="Cancelar"
                  locale="es"
                  selectText="Seleccionar"
                  v-model="form.pago_fechas"
                  multi-dates
                  :min-date="new Date()"
                  placeholder="Selecciona fechas de pago"
                  teleport-center
                  @cleared="form.pago_fechas = []"
                />
              </v-col>
              <v-col cols="12">
                <h6 class="mb-3 text-h6">Marca la ubicación de cobro</h6>
                <MapsComponent
                  @ubicacion="
                    (ubi) => {
                      form.ubicacionMap = ubi;
                    }
                  "
                />
              </v-col>
            </v-row>
          </v-form>
        </v-card-text>
        <v-card-actions class="justify-end">
          <v-btn
            color="red-darken-1"
            variant="tonal"
            @click="dialogPrestamo = false"
          >
            Cerrar
          </v-btn>
          <v-btn
            color="green-darken-1"
            variant="tonal"
            :disabled="disableBtn"
            @click="guardar"
          >
            Crear
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <nuevoCliente
      :dialogCliente="dialogCliente"
      @cerrarDialog="dialogCliente = false"
      @actualizarTodo="actualizarTodo"
    />
    <v-dialog v-model="dialogUbicacion" persistent width="700">
      <v-card>
        <v-card-title> Cambiar ubicación </v-card-title>
        <v-card-text>
          <v-col cols="12">
            <h6 class="mb-3 text-h6">Marca la ubicación de cobro</h6>
            <MapsComponent
              :centroMap="actualizarUbicacion.ubicacionMap"
              @ubicacion="
                (ubi) => {
                  actualizarUbicacion.ubicacionMap = ubi;
                }
              "
            />
          </v-col>
        </v-card-text>
        <v-card-actions class="justify-end">
          <v-btn
            color="red-darken-1"
            variant="tonal"
            @click="dialogUbicacion = false"
          >
            Cerrar
          </v-btn>
          <v-btn
            color="green-darken-1"
            variant="tonal"
            :disabled="disableBtn"
            @click="cambiarUbicacion"
          >
            Cambiar ubicación
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import Session from "@/validation/session";
import Chart from "chart.js/auto";
import nuevoCliente from "@/components/nuevoCliente.vue";
import axios from "axios";
import Swal from "sweetalert2";
export default {
  name: "cobroVista",
  components: { nuevoCliente },
  data: () => ({
    api: import.meta.env.VITE_APP_API_URL,
    loadingTable: false,
    disableBtn: false,
    fechaAbonoApi: [],
    cobros: [],
    cobroRutaGrafica: null,
    abonosHoy: [],
    abonosFechaEspecifica: [],
    valid: false,
    token: null,
    dialogCliente: false,
    dialogAbonar: null,
    dialogVerCobro: null,
    dialogPrestamo: null,
    dialogUbicacion: false,
    searchPrestamo: null,
    cedulaTemp: null,
    montoSugerido: null,
    clientes: [],
    productos: [],
    formaPago: null,
    form: {
      cliente: null,
      ruta: null,
      producto: null,
      fecha_inicio: "",
      cantidad: 1,
      cuotas: 1,
      pago_fechas: [],
      total: 0,
      ubicacionMap: {
        lat: null,
        lng: null,
      },
    },
    formAbono: {
      id: null,
      abono: null,
    },
    actualizarUbicacion: {
      venta: null,
      ubicacionMap: {
        lat: null,
        lng: null,
      },
    },
    verPrestamo: {
      abono: null,
      mora: null,
      _id: null,
      cliente: {
        _id: null,
        documento: null,
        nombres: null,
        apellidos: null,
        telefono: null,
        correo: null,
        direccion: null,
        mora: null,
        __v: null,
      },
      ruta: null,
      producto: null,
      fecha_inicio: null,
      cuotas: null,
      pago_fechas: [
        {
          fecha: null,
          monto: null,
        },
      ],
      cuotas_atrasadas: null,
      completado: null,
      total: null,
    },
    formasPago: [
      { index: 1, forma: "De contado" },
      { index: 2, forma: "A crédito" },
    ],
    campoRules: [(v) => !!v || "Campo requerido"],
    cantidadRules: [
      (v) => !!v || "Campo requerido",
      (v) => parseInt(v) > 0 || "Ingrese una cantidad mayor a 0",
    ],
    headers: [
      { title: "Documento", key: "cliente.documento" },
      { title: "Nombre", key: "cliente.nombres" },
      { title: "Apellido", key: "cliente.apellidos" },
      { title: "Ruta", key: "cliente.direccion.nombre" },
      { title: "Producto", key: "producto" },
      { title: "Cuotas", key: "cuotas" },
      { title: "Atrasado", key: "mora" },
      { title: "Accion", key: "actions", sortable: false },
    ],
    headersAbonoHoy: [
      { title: "Documento", key: "cliente.documento" },
      { title: "Nombre", key: "cliente.nombres" },
      { title: "Apellido", key: "cliente.apellidos" },
      { title: "Ruta", key: "cliente.direccion.nombre" },
      { title: "Cobrador", key: "cobrador" },
      { title: "Pago", key: "monto" },
    ],
  }),
  methods: {
    dialogActualizarUbicacion(id) {
      if (id != null) {
        this.actualizarUbicacion.venta = id;
        this.dialogUbicacion = true;
      }
    },
    calcularAbono(abono) {
      let abonoTotal = 0;
      abono.forEach((abono) => {
        abonoTotal += abono.monto;
      });
      return abonoTotal.toLocaleString();
    },
    calcularRestante(abono, total) {
      let abonoTotal = 0;
      abono.forEach((abono) => {
        abonoTotal += abono.monto;
      });
      return (total - abonoTotal).toLocaleString();
    },
    formatDate(value) {
      const date = new Date(value);
      const year = date.getFullYear();
      const month = (date.getMonth() + 1).toString().padStart(2, "0");
      const day = date.getDate().toString().padStart(2, "0");
      const hours = date.getHours().toString().padStart(2, "0");
      const minutes = date.getMinutes().toString().padStart(2, "0");
      const seconds = date.getSeconds().toString().padStart(2, "0");

      return `${year}-${month}-${day} | ${hours}:${minutes}:${seconds}`;
    },
    async cambiarUbicacion() {
      if (
        this.actualizarUbicacion.venta != null &&
        this.actualizarUbicacion.ubicacionMap.lat != null &&
        this.actualizarUbicacion.ubicacionMap.lng != null
      ) {
        await axios
          .put(`${this.api}/prestamo/ubicacion`, this.actualizarUbicacion, {
            headers: {
              Authorization: `Bearer ${this.token}`,
            },
          })
          .then(() => {
            Swal.fire({
              icon: "success",
              text: "Ubicación actualizada correctamente",
              showConfirmButton: false,
              timer: 1500,
            });
            this.dialogUbicacion = false;
            this.actualizarTodo();
          })
          .catch((error) => {
            switch (error.response.status) {
              case 401:
                Session.expiredSession();
                break;
              default:
                Swal.fire({
                  icon: "info",
                  text: "No se pudo actualizar la ubicación",
                  showConfirmButton: false,
                  timer: 1600,
                });
                break;
            }
          });
      } else {
        Swal.fire({
          icon: "error",
          title: "Ubicación",
          text: "Debe seleccionar una ubicación en el mapa",
          showConfirmButton: false,
          timer: 1600,
        });
      }
    },
    async actualizarTodo() {
      await this.obtenerCobros();
      await this.getProductosInventario();
      await this.getClientes();
    },
    async obtenerCobros() {
      this.loadingTable = true;
      await axios
        .get(`${this.api}/prestamo/cobrar`, {
          headers: {
            Authorization: `Bearer ${this.token}`,
          },
        })
        .then((resp) => {
          this.cobros = resp.data[0];
          this.abonosHoy = resp.data[1];
          const data_cobro_ruta = resp.data[2];
          let ctx = document.getElementById("graficaAbonoRutas");
          const cobro_ruta_bar = {
            type: "bar",
            data: {
              labels: data_cobro_ruta.labels,
              datasets: [
                {
                  label: "Abonos",
                  data: data_cobro_ruta.data,
                  borderWidth: 1,
                  backgroundColor: this.colores,
                },
              ],
            },
            options: {
              plugins: {
                title: {
                  display: true,
                  text: "Abonos Por Ruta",
                },
                legend: {
                  display: false,
                },
              },
              scales: {
                y: {
                  beginAtZero: true,
                },
              },
            },
          };
          this.cobroRutaGrafica = new Chart(ctx, cobro_ruta_bar);
        })
        .catch((error) => {
          switch (error.response.status) {
            case 401:
              Session.expiredSession();
              break;
            default:
              Swal.fire({
                icon: "info",
                text: "No se pudo obtener los cobros",
                showConfirmButton: false,
                timer: 1600,
              });
              break;
          }
        });
      this.loadingTable = false;
    },
    verPrestamoFunction(item) {
      this.verPrestamo = item;
      this.verPrestamo.fecha_inicio = this.formatDate(
        this.verPrestamo.fecha_inicio
      );
      this.dialogVerCobro = true;
    },
    abonarFunction(item) {
      this.formAbono.id = item._id;
      this.cedulaTemp = item.cliente.documento;
      this.montoSugerido = item.cuota_sugerida;
      this.dialogAbonar = true;
    },
    async abonar() {
      const { valid } = await this.$refs.formAbono.validate();
      if (valid) {
        this.disableBtn = true;
        this.dialogAbonar = false;
        this.formAbono.abono = parseInt(this.formAbono.abono);
        await axios
          .post(`${this.api}/prestamo/cobrar`, this.formAbono, {
            headers: {
              Authorization: `Bearer ${this.token}`,
            },
          })
          .then(() => {
            return Swal.fire({
              icon: "success",
              title: "Exitoso",
              text: "Abonado correctamente!",
              showConfirmButton: false,
              timer: 1500,
            });
          })
          .catch((error) => {
            console.log(error);
            return Swal.fire({
              icon: "error",
              title: "No se pudo abonar",
              showConfirmButton: false,
              timer: 1500,
            });
          });
        this.formAbono = {
          id: null,
          abono: null,
        };
        this.disableBtn = false;
        await this.obtenerCobros();
      }
    },
    async getClientes() {
      await axios
        .get(`${this.api}/prestamo/clientes`, {
          headers: {
            Authorization: `Bearer ${this.token}`,
          },
        })
        .then((resp) => {
          this.clientes = resp.data;
        })
        .catch((error) => {
          switch (error.response.status) {
            case 401:
              Session.expiredSession();
              break;
            default:
              Swal.fire({
                icon: "info",
                text: "No se pudo obtener los clientes",
                showConfirmButton: false,
                timer: 1600,
              });
              break;
          }
        });
    },
    async getProductosInventario() {
      await axios
        .get(`${this.api}/inventario/existe`, {
          headers: {
            Authorization: `Bearer ${this.token}`,
          },
        })
        .then((resp) => {
          this.productos = resp.data;
        })
        .catch((error) => {
          switch (error.response.status) {
            case 401:
              Session.expiredSession();
              break;
            default:
              Swal.fire({
                icon: "info",
                text: "No se pudo obtener los productos",
                showConfirmButton: false,
                timer: 1600,
              });
              break;
          }
        });
    },
    async guardar() {
      const { valid } = await this.$refs.formPrestamo.validate();
      if (valid) {
        if (
          this.form.ubicacionMap.lat == null ||
          this.form.ubicacionMap.lng == null
        ) {
          Swal.fire({
            icon: "error",
            title: "Ubicación",
            text: "Debe seleccionar una ubicación en el mapa",
            showConfirmButton: false,
            timer: 1600,
          });
          return;
        }
        this.form.cantidad = parseInt(this.form.cantidad);
        this.form.cuotas = parseInt(this.form.cuotas);
        let total,
          pagos = [];
        if (
          this.formaPago == 2 &&
          this.form.pago_fechas.length != this.form.cuotas
        ) {
          return Swal.fire({
            icon: "warning",
            text: "Las fechas de pago deben ser iguales a la cantidad de cuotas",
            showConfirmButton: false,
            timer: 1690,
          });
        }
        if (this.form.cantidad <= this.form.producto.existencias) {
          this.disableBtn = true;
          if (this.form.pago_fechas.length > 1) {
            this.form.pago_fechas = this.ordenarFechas(this.form.pago_fechas);
          }
          if (parseInt(this.formaPago) == 1) {
            total =
              this.form.cantidad * this.form.producto.producto.valor_contado;
            this.form.cuotas = 0;
          } else {
            total =
              this.form.cantidad * this.form.producto.producto.valor_credito;
            const cuotaBase = Math.ceil(total / this.form.cuotas); //Redondeo hacia arriba
            const valorCuota = Math.floor(cuotaBase / 100) * 100; //Redondeo hacia abajo, con parte entera de 100
            const restVal = Math.round(
              (cuotaBase - valorCuota) * this.form.cuotas
            ); //Diferencia, con redondeo al más cercano
            pagos = this.form.pago_fechas.map((pago, index) => {
              return index == 0
                ? { fecha: pago, monto: valorCuota + restVal }
                : { fecha: pago, monto: valorCuota };
            });
          }
          const paquete = {
            cliente: this.form.cliente._id, //`${this.form.cliente.nombres} ${this.form.cliente.apellidos}`,
            ruta: this.form.cliente.direccion.nombre,
            producto: this.form.producto.producto.nombre, //ObjectId del inventario
            inventario: this.form.producto._id,
            fecha_inicio: new Date().toISOString(),
            cantidad: this.form.cantidad,
            cuotas: this.form.cuotas,
            pago_fechas: pagos,
            total: total,
            ubicacionMap: this.form.ubicacionMap,
          };
          await axios
            .post(`${this.api}/prestamo/crear`, paquete, {
              headers: {
                Authorization: `Bearer ${this.token}`,
              },
            })
            .then(() => {
              this.form.pago_fechas = [];
              this.$refs.formPrestamo.reset();
              Swal.fire({
                icon: "success",
                text: "Venta registrada correctamente",
                showConfirmButton: false,
                timer: 1710,
              });
              this.dialogPrestamo = false;
            })
            .catch((error) => {
              switch (error.response.status) {
                case 401:
                  Session.expiredSession();
                  break;
                default:
                  Swal.fire({
                    icon: "info",
                    text: "No se pudo crear la venta",
                    showConfirmButton: false,
                    timer: 1710,
                  });
                  break;
              }
            });
        } else {
          this.disableBtn = false;
          return Swal.fire({
            icon: "warning",
            text: `No hay suficientes ${this.form.producto.producto.nombre}, sólo hay ${this.form.producto.existencias}`,
            showConfirmButton: false,
            timer: 1600,
          });
        }
      }
      this.disableBtn = false;
      await this.actualizarTodo();
    },
    ordenarFechas(fechas = []) {
      const fechasDate = fechas.map((fecha) => new Date(fecha));
      fechasDate.sort((f1, f2) => f1 - f2);
      return fechasDate;
    },
    async findCobroEspecifico() {
      if (this.fechaAbonoApi) {
        this.$emit("loadingSweet");
        await axios
          .post(
            `${this.api}/prestamo/cobroEspecifico`,
            {
              date: this.fechaAbonoApi,
            },
            {
              headers: {
                Authorization: `Bearer ${this.token}`,
              },
            }
          )
          .then((resp) => {
            this.abonosFechaEspecifica = resp.data;
          })
          .catch((error) => {
            switch (error.response.status) {
              case 401:
                Session.expiredSession();
                break;
              default:
                Swal.fire({
                  icon: "info",
                  text: "No se pudo obtener los productos",
                  showConfirmButton: false,
                  timer: 1600,
                });
                break;
            }
          });
        this.$emit("closeSweet");
      }
    },
    calcularAbonoTotales(abonos) {
      let montoT = 0;
      abonos.forEach((abono) => {
        montoT += abono.monto;
      });
      return montoT.toLocaleString();
    },
  },
  computed: {
    cols2() {
      //Para que los input del modal sean responsive
      const { xxl, xl, lg, md } = this.$vuetify.display;
      let resp = [12, 12];
      if (xxl || xl || lg || md) {
        resp = [6, 6];
      }
      return resp;
    },
    skill() {
      let abonado = 0;
      this.verPrestamo.abono.forEach((abono) => {
        abonado += abono.monto;
      });
      if (this.verPrestamo.cuotas == 0) {
        return 100;
      }
      return (abonado * 100) / this.verPrestamo.total;
    },
  },
  async created() {
    const invalid = await Session.expiredSession();
    if (!invalid) {
      this.token = this.$store.getters.usuario.usuario.access_token;
      this.$emit("loadingSweet");
      await this.obtenerCobros();
      await this.getProductosInventario();
      await this.getClientes();
      this.$emit("closeSweet");
    }
  },
};
</script>
