<template>
  <q-layout view="hHh lpR fFf" class="app-layout">

    <!-- HEADER -->
    <q-header elevated class="header-taller">
      <q-toolbar class="header-content">
        <div class="brand">
          <div>
            <div class="text-h5 text-weight-bold">Efraín Tech Service</div>
            <div class="text-caption text-blue-1">
              Servicio técnico de celulares y tablets
            </div>
          </div>

          <img
            src="./img/IMG_1192.PNG"
            alt="Logo Efraín Tech Service"
            class="logo"
          />
        </div>

        <q-space />

        <q-input
          v-model="textoBusqueda"
          dense
          outlined
          clearable
          bg-color="white"
          placeholder="Buscar cliente, marca o modelo..."
          class="search"
        >
          <template #prepend>
            <q-icon name="search" color="grey-7" />
          </template>
        </q-input>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="page-content q-pa-md">

        <!-- RESUMEN -->
        <div class="row q-col-gutter-md q-mb-md">
          <div
            v-for="item in resumen"
            :key="item.label"
            class="col-6 col-sm-3"
          >
            <q-card flat class="stat-card">
              <q-card-section class="text-center q-pa-sm">
                <div class="text-caption text-grey-7">{{ item.label }}</div>
                <div :class="`text-h5 text-weight-bold text-${item.color}`">
                  {{ item.valor() }}
                </div>
              </q-card-section>
            </q-card>
          </div>
        </div>

        <!-- CONTROLES -->
        <div class="control-bar row items-center justify-between q-pa-sm q-mb-md">
          <div class="text-subtitle1 text-weight-bold">
            Servicios Registrados ({{ serviciosFiltrados.length }})
          </div>

          <q-select
            v-model="filtroEstado"
            :options="opcionesFiltroEtapas"
            emit-value
            map-options
            dense
            outlined
            bg-color="white"
            label="Etapa de trabajo"
            style="min-width:220px"
          />
        </div>

        <!-- SERVICIOS -->
        <div class="row q-col-gutter-md">
          <div
            v-for="servicio in serviciosFiltrados"
            :key="servicio.id"
            class="col-12 col-sm-6 col-md-4"
          >
            <q-card flat class="device-card">

              <div
                v-if="servicio.estadoPago === 'pendiente'"
                class="payment pending"
              >
                PAGO PENDIENTE
              </div>

              <div
                v-else-if="servicio.estadoPago === 'abono'"
                class="payment abonado"
              >
                ABONADO — Falta ${{ formatearNumero(calcularSaldo(servicio)) }}
              </div>

              <q-card-section>
                <div class="row items-center justify-between">
                  <div>
                    <div class="text-subtitle1 text-weight-bold">
                      {{ servicio.cliente }}
                    </div>
                    <div class="text-primary text-weight-medium">
                      {{ servicio.marca }} {{ servicio.modelo }}
                    </div>
                  </div>

                  <q-badge color="grey-3" text-color="grey-9">
                    {{ etiquetaReparacion(servicio.tipoReparacion) }}
                  </q-badge>
                </div>
              </q-card-section>

              <q-separator />

              <q-card-section class="text-caption">

                <div class="row justify-between q-mb-sm">
                  <span class="text-grey-7">Etapa:</span>
                  <q-badge
                    :color="colorEstado(servicio.estadoEquipo)"
                    class="text-weight-bold"
                  >
                    {{ etiquetaEstado(servicio.estadoEquipo) }}
                  </q-badge>
                </div>

                <div class="row justify-between text-grey-8">
                  <span>Técnico: {{ servicio.tecnico }}</span>
                  <span>{{ formatearFecha(servicio.fechaRecepcion) }}</span>
                </div>

                <div class="row justify-between q-mt-sm text-subtitle2">
                  <span>
                    Precio:
                    <strong>${{ formatearNumero(servicio.precio) }}</strong>
                  </span>
                  <span class="text-caption text-grey-7">
                    {{ servicio.metodoPago || '—' }}
                  </span>
                </div>

                <div
                  v-if="servicio.observaciones"
                  class="text-grey-7 text-italic q-mt-sm"
                >
                  Obs: {{ servicio.observaciones }}
                </div>

                <div
                  v-if="servicio.estadoEquipo === 'entregado'"
                  class="row items-center justify-between q-mt-sm"
                >
                  <span>Calificación:</span>

                  <q-rating
                    v-model="servicio.calificacion"
                    size="1.2em"
                    color="amber"
                    icon="star_border"
                    icon-selected="star"
                    @update:model-value="guardarCambiosDirectos(servicio)"
                  />
                </div>

              </q-card-section>

              <q-separator />

              <q-card-actions align="right">
                <q-btn
                  flat dense
                  icon="edit"
                  color="primary"
                  label="Editar"
                  no-caps
                  @click="abrirModalEditar(servicio)"
                />

                <q-btn
                  flat dense
                  icon="delete"
                  color="negative"
                  label="Eliminar"
                  no-caps
                  @click="abrirConfirmarEliminar(servicio)"
                />
              </q-card-actions>

            </q-card>
          </div>

          <!-- SIN RESULTADOS -->
          <div
            v-if="!serviciosFiltrados.length"
            class="col-12 text-center q-pa-xl empty-box"
          >
            <div class="text-subtitle1 text-grey-8">
              No hay servicios registrados en esta sección
            </div>

            <q-btn
              v-if="textoBusqueda || filtroEstado !== 'todos'"
              flat
              color="primary"
              label="Limpiar filtros"
              no-caps
              class="q-mt-sm"
              @click="limpiarFiltros"
            />
          </div>
        </div>

        <!-- BOTÓN NUEVO -->
        <q-page-sticky position="bottom-right" :offset="[18,18]">
          <q-btn
            fab
            icon="add"
            color="primary"
            @click="abrirModalNuevo"
          />
        </q-page-sticky>

      </q-page>
    </q-page-container>

    <!-- FORMULARIO -->
    <q-dialog v-model="mostrarModalFormulario" persistent>
      <q-card class="modal">

        <q-card-section class="row items-center bg-primary text-white">
          <div class="text-h6 text-weight-bold">
            {{ modoEdicion ? 'Editar servicio' : 'Nuevo servicio' }}
          </div>

          <q-space />

          <q-btn
            flat round dense
            icon="close"
            @click="cerrarModalFormulario"
          />
        </q-card-section>

        <q-card-section class="scroll form-body">
          <q-form ref="formularioRef">
            <div class="row q-col-gutter-md">

              <div class="col-12 col-sm-6">
                <q-input
                  v-model="formulario.cliente"
                  outlined dense
                  label="Nombre del cliente *"
                  :rules="[requerido]"
                />
              </div>

              <div class="col-6 col-sm-3">
                <q-input
                  v-model="formulario.marca"
                  outlined dense
                  label="Marca *"
                  :rules="[requerido]"
                />
              </div>

              <div class="col-6 col-sm-3">
                <q-input
                  v-model="formulario.modelo"
                  outlined dense
                  label="Modelo *"
                  :rules="[requerido]"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-select
                  v-model="formulario.tipoReparacion"
                  :options="opcionesTipoReparacion"
                  emit-value
                  map-options
                  outlined dense
                  label="Tipo de reparación *"
                  :rules="[requerido]"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-select
                  v-model="formulario.tecnico"
                  :options="opcionesTecnicos"
                  outlined dense
                  label="Técnico *"
                  :rules="[requerido]"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-input
                  v-model="formulario.fechaRecepcion"
                  type="datetime-local"
                  outlined dense
                  stack-label
                  label="Fecha de recepción *"
                  :rules="[requerido]"
                />
              </div>

              <div class="col-6 col-sm-3">
                <q-input
                  v-model.number="formulario.precio"
                  type="number"
                  outlined dense
                  prefix="$"
                  label="Precio *"
                  :rules="[val => val >= 0 || 'Precio inválido']"
                />
              </div>

              <div class="col-6 col-sm-3">
                <q-select
                  v-model="formulario.metodoPago"
                  :options="opcionesMetodoPago"
                  outlined dense
                  label="Método de pago *"
                  :rules="[requerido]"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-select
                  v-model="formulario.estadoPago"
                  :options="opcionesEstadoPago"
                  outlined dense
                  label="Estado del pago *"
                  :rules="[requerido]"
                />
              </div>

              <div
                v-if="formulario.estadoPago === 'abono'"
                class="col-12 col-sm-6"
              >
                <q-input
                  v-model.number="formulario.montoAbonado"
                  type="number"
                  outlined dense
                  prefix="$"
                  label="Monto abonado"
                  :rules="[
                    val => val >= 0 || 'Valor inválido',
                    val => val <= formulario.precio || 'No puede superar el precio'
                  ]"
                />
              </div>

              <div class="col-12">
                <q-select
                  v-model="formulario.estadoEquipo"
                  :options="opcionesEstadoEquipo"
                  emit-value
                  map-options
                  outlined dense
                  label="Estado del equipo *"
                  :rules="[requerido]"
                />
              </div>

              <div class="col-12">
                <q-input
                  v-model="formulario.observaciones"
                  type="textarea"
                  autogrow
                  outlined
                  label="Observaciones"
                />
              </div>

            </div>
          </q-form>
        </q-card-section>

        <q-card-actions align="right" class="bg-grey-1">
          <q-btn
            flat
            label="Cancelar"
            color="grey-8"
            @click="cerrarModalFormulario"
          />

          <q-btn
            unelevated
            label="Guardar"
            color="primary"
            icon="save"
            @click="guardarServicio"
          />
        </q-card-actions>

      </q-card>
    </q-dialog>

    <!-- ELIMINAR -->
    <q-dialog v-model="mostrarModalEliminar">
      <q-card class="delete-modal">

        <q-card-section class="bg-negative text-white">
          <div class="text-h6 text-weight-bold">
            ¿Eliminar servicio?
          </div>
        </q-card-section>

        <q-card-section>
          Vas a eliminar el registro de
          <strong>{{ servicioAEliminar?.cliente }}</strong>
          ({{ servicioAEliminar?.marca }}
          {{ servicioAEliminar?.modelo }}).
          <br><br>
          Esta acción no se puede deshacer.
        </q-card-section>

        <q-card-actions align="right">
          <q-btn
            flat
            label="Cancelar"
            @click="mostrarModalEliminar = false"
          />

          <q-btn
            unelevated
            color="negative"
            icon="delete"
            label="Sí, eliminar"
            @click="eliminarServicio"
          />
        </q-card-actions>

      </q-card>
    </q-dialog>

    <!-- AVISO -->
    <q-banner
      v-if="mensajeAviso"
      class="fixed-bottom text-white text-center text-weight-bold"
      :class="mensajeAvisoTipo === 'error' ? 'bg-negative' : 'bg-positive'"
      style="z-index:9999"
    >
      {{ mensajeAviso }}
    </q-banner>

  </q-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const opcionesTipoReparacion = [
  { label:'Cambio de pantalla', value:'pantalla' },
  { label:'Cambio de batería', value:'bateria' },
  { label:'Cambio de pin de carga', value:'pin_carga' },
  { label:'Liberación', value:'liberacion' },
  { label:'Mantenimiento de software', value:'software' },
  { label:'Cambio de flex', value:'flex' },
  { label:'Otros', value:'otros' }
]

const opcionesTecnicos = [
  'Efraín Sarmiento Vesga',
  'Helver Remolina Rondon',
  'Sergio Aguirre Baez'
]

const opcionesMetodoPago = [
  'Efectivo',
  'Transferencia',
  'Tarjeta'
]

const opcionesEstadoPago = [
  { label:'Pagado', value:'pagado' },
  { label:'Pendiente', value:'pendiente' },
  { label:'Abono', value:'abono' }
]

const opcionesEstadoEquipo = [
  { label:'Recibido', value:'recibido' },
  { label:'En reparación', value:'en_reparacion' },
  { label:'Listo para entregar', value:'listo' },
  { label:'Entregado', value:'entregado' }
]

const opcionesFiltroEtapas = [
  { label:'Todas las etapas', value:'todos' },
  ...opcionesEstadoEquipo.map(x => ({
    label:x.label + (x.value === 'entregado' ? 's' : 's'),
    value:x.value
  }))
]

const servicios = ref(
  JSON.parse(localStorage.getItem('taller-don-efrain-servicios') || '[]')
)

const textoBusqueda = ref('')
const filtroEstado = ref('todos')

const mostrarModalFormulario = ref(false)
const mostrarModalEliminar = ref(false)
const modoEdicion = ref(false)
const idEnEdicion = ref(null)
const servicioAEliminar = ref(null)
const formularioRef = ref(null)

const mensajeAviso = ref('')
const mensajeAvisoTipo = ref('exito')

const formularioVacio = () => ({
  cliente:'',
  marca:'',
  modelo:'',
  tipoReparacion:null,
  tecnico:null,
  fechaRecepcion:'',
  precio:0,
  metodoPago:null,
  estadoPago:null,
  montoAbonado:0,
  estadoEquipo:'recibido',
  calificacion:0,
  observaciones:''
})

const formulario = ref(formularioVacio())

const guardarDatos = () => {
  localStorage.setItem(
    'taller-don-efrain-servicios',
    JSON.stringify(servicios.value)
  )
}

const requerido = val =>
  !!String(val || '').trim() || 'Campo obligatorio'

const serviciosFiltrados = computed(() => {
  const texto = textoBusqueda.value.trim().toLowerCase()

  return servicios.value.filter(s =>
    (filtroEstado.value === 'todos' ||
      s.estadoEquipo === filtroEstado.value) &&
    (!texto ||
      [s.cliente, s.marca, s.modelo]
        .some(x => String(x || '').toLowerCase().includes(texto)))
  )
})

const resumen = [
  {
    label:'En taller',
    color:'primary',
    valor:() =>
      contar('recibido') + contar('en_reparacion')
  },
  {
    label:'Listos para entregar',
    color:'positive',
    valor:() => contar('listo')
  },
  {
    label:'Pagos pendientes',
    color:'negative',
    valor:() =>
      servicios.value.filter(s => s.estadoPago === 'pendiente').length
  },
  {
    label:'Total fiado',
    color:'teal-8',
    valor:() => '$' + formatearNumero(totalFiado())
  }
]

function contar(estado) {
  return servicios.value.filter(
    s => s.estadoEquipo === estado
  ).length
}

function totalFiado() {
  return servicios.value
    .filter(s => s.estadoPago === 'abono')
    .reduce((total, s) => total + calcularSaldo(s), 0)
}

function calcularSaldo(s) {
  return Math.max(
    (Number(s.precio) || 0) -
    (Number(s.montoAbonado) || 0),
    0
  )
}

function formatearNumero(valor) {
  return (Number(valor) || 0).toLocaleString('es-CO')
}

function formatearFecha(fecha) {
  if (!fecha) return 'Sin fecha'

  const f = new Date(fecha)
  if (isNaN(f)) return 'Fecha inválida'

  return f.toLocaleString('es-CO', {
    day:'2-digit',
    month:'short',
    year:'numeric',
    hour:'2-digit',
    minute:'2-digit'
  })
}

function obtenerFechaHoraActual() {
  const d = new Date()
  d.setMinutes(d.getMinutes() - d.getTimezoneOffset())
  return d.toISOString().slice(0,16)
}

function etiquetaReparacion(valor) {
  return opcionesTipoReparacion.find(
    x => x.value === valor
  )?.label || valor || 'Sin especificar'
}

function etiquetaEstado(estado) {
  return opcionesEstadoEquipo.find(
    x => x.value === estado
  )?.label || 'Sin estado'
}

function colorEstado(estado) {
  return {
    recibido:'blue-grey',
    en_reparacion:'orange-9',
    listo:'positive',
    entregado:'grey-7'
  }[estado] || 'grey'
}

function abrirModalNuevo() {
  modoEdicion.value = false
  idEnEdicion.value = null
  formulario.value = formularioVacio()
  formulario.value.fechaRecepcion = obtenerFechaHoraActual()
  mostrarModalFormulario.value = true
}

function abrirModalEditar(servicio) {
  modoEdicion.value = true
  idEnEdicion.value = servicio.id
  formulario.value = { ...servicio }
  mostrarModalFormulario.value = true
}

function cerrarModalFormulario() {
  mostrarModalFormulario.value = false
  formulario.value = formularioVacio()
  modoEdicion.value = false
  idEnEdicion.value = null
}

async function guardarServicio() {
  const valido = await formularioRef.value.validate()

  if (!valido) {
    return mostrarAviso(
      'Revisa los campos marcados en rojo',
      'error'
    )
  }

  const precio = Number(formulario.value.precio) || 0
  const abonado = Number(formulario.value.montoAbonado) || 0

  if (
    formulario.value.estadoPago === 'abono' &&
    abonado > precio
  ) {
    return mostrarAviso(
      'El abono no puede superar el precio',
      'error'
    )
  }

  formulario.value.precio = precio

  if (formulario.value.estadoPago === 'pagado')
    formulario.value.montoAbonado = precio

  if (formulario.value.estadoPago === 'pendiente')
    formulario.value.montoAbonado = 0

  if (modoEdicion.value) {
    const i = servicios.value.findIndex(
      s => s.id === idEnEdicion.value
    )

    if (i === -1)
      return mostrarAviso('Servicio no encontrado', 'error')

    servicios.value[i] = {
      ...formulario.value,
      id:idEnEdicion.value
    }

    mostrarAviso('Servicio actualizado correctamente')
  } else {
    servicios.value.unshift({
      ...formulario.value,
      id:Date.now().toString()
    })

    mostrarAviso('Servicio registrado correctamente')
  }

  guardarDatos()
  cerrarModalFormulario()
}

function guardarCambiosDirectos(servicio) {
  guardarDatos()
  mostrarAviso('Calificación guardada')
}

function abrirConfirmarEliminar(servicio) {
  servicioAEliminar.value = servicio
  mostrarModalEliminar.value = true
}

function eliminarServicio() {
  if (!servicioAEliminar.value) return

  servicios.value = servicios.value.filter(
    s => s.id !== servicioAEliminar.value.id
  )

  guardarDatos()
  mostrarModalEliminar.value = false
  servicioAEliminar.value = null
  mostrarAviso('Servicio eliminado')
}

function limpiarFiltros() {
  textoBusqueda.value = ''
  filtroEstado.value = 'todos'
}

let temporizadorAviso

function mostrarAviso(texto, tipo = 'exito') {
  mensajeAviso.value = texto
  mensajeAvisoTipo.value = tipo

  clearTimeout(temporizadorAviso)

  temporizadorAviso = setTimeout(() => {
    mensajeAviso.value = ''
  }, 2500)
}
</script>

<style scoped>
.app-layout {
  background:#f8fafc;
  color:#1e293b;
  font-family:system-ui,-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,sans-serif;
}

.header-taller {
  background:#095de4;
  padding: 10px 20%;
}

.header-content {
  padding:10px 24px;
  gap:24px;
}

.brand {
  display:flex;
  align-items:center;
  gap:20px;
}

.logo {
  width:90px;
  max-height:70px;
  object-fit:contain;
}

.search {
  width:320px;
  max-width:40vw;
}

.page-content {
  min-height:100vh;
  background:#f8fafc;
}

.stat-card,
.control-bar,
.device-card,
.empty-box {
  background:#fff;
  border:1px solid #e2e8f0;
  border-radius:12px;
}

.device-card {
  box-shadow:0 1px 3px rgba(0,0,0,.05);
  overflow:hidden;
}

.payment {
  padding:5px;
  text-align:center;
  color:#fff;
  font-size:12px;
  font-weight:bold;
}

.pending {
  background:#c10015;
}

.abonado {
  background:#f57c00;
}

.modal {
  width:650px;
  max-width:95vw;
  border-radius:12px;
  overflow:hidden;
}

.form-body {
  max-height:70vh;
}

.delete-modal {
  width:400px;
  max-width:90vw;
}

@media (max-width:700px) {
  .header-content {
    flex-wrap:wrap;
  }

  .brand {
    width:100%;
    justify-content:space-between;
  }

  .search {
    width:100%;
    max-width:none;
  }

  .logo {
    width:70px;
  }
}
</style>
