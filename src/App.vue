<template>
  <q-layout view="hHh lpR fFf" class="app-layout">

    <q-header elevated class="header-taller">
      <q-toolbar class="header-content">
        <div class="brand">
          <div>
            <div class="text-h5 text-weight-bold">Efraín Tech Service</div>
            <div class="text-caption text-blue-1">
              Servicio técnico de celulares y tablets
            </div>
          </div>

          <img src="./img/IMG_1192.PNG" alt="Logo Efraín Tech Service" class="logo" />
        </div>

        <q-space />

        <q-input v-model="textoBusqueda" dense outlined clearable bg-color="white"
          placeholder="Buscar cliente, marca o modelo..." class="search">
          <template #prepend>
            <q-icon name="search" color="grey-7" />
          </template>
        </q-input>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="page-content q-pa-md">
        <div class="row q-col-gutter-md q-mb-md">
          <div v-for="item in resumen" :key="item.label" class="col-6 col-sm-3">
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
        <div class="control-bar row items-center justify-between q-pa-sm q-mb-md">
          <div class="text-subtitle1 text-weight-bold">
            Servicios Registrados ({{ serviciosFiltrados.length }})
          </div>
          <q-select v-model="filtroEstado" :options="opcionesFiltroEtapas" emit-value map-options dense outlined
            bg-color="white" label="Etapa de trabajo" style="min-width:220px" />
        </div>

        <div class="row q-col-gutter-md">
          <div v-for="servicio in serviciosFiltrados" :key="servicio.id" class="col-12 col-sm-6 col-md-4">
            <q-card flat class="device-card">

              <div v-if="servicio.estadoPago === 'pendiente'" class="payment pending">
                PAGO PENDIENTE
              </div>

              <div v-else-if="servicio.estadoPago === 'abono'" class="payment abonado">
                ABONADO — Falta ${{ formatearNumero(calcularSaldo(servicio)) }}
              </div>

              <q-card-section>
                <div class="row items-center justify-between">
                  <div>
                    <div class="text-subtitle1 text-weight-bold">
                      {{ servicio.cliente }}
                    </div>
                    <div class="text-primary text-weight-medium marca-modelo">
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
                  <q-badge :color="colorEstado(servicio.estadoEquipo)" class="text-weight-bold">
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

                <div v-if="servicio.observaciones" class="text-grey-7 text-italic q-mt-sm">
                  Obs: {{ servicio.observaciones }}
                </div>
                <div v-if="servicio.estadoEquipo === 'entregado'" class="row items-center justify-between q-mt-sm">
                  <span>Calificación:</span>

                  <q-rating v-model="servicio.calificacion" size="1.3em" color="amber" icon="star_border"
                    icon-selected="star" @update:model-value="guardarCambiosDirectos(servicio)" />
                </div>

              </q-card-section>

              <q-separator />

              <!-- Un servicio entregado ya no se puede editar ni eliminar -->
              <q-card-actions v-if="servicio.estadoEquipo !== 'entregado'" align="right">
                <q-btn flat dense icon="edit" color="primary" label="Editar" no-caps
                  @click="abrirModalEditar(servicio)" />

                <q-btn flat dense icon="delete" color="negative" label="Eliminar" no-caps
                  @click="abrirConfirmarEliminar(servicio)" />
              </q-card-actions>

              <q-card-section v-else class="text-center entregado-lock">
                <q-icon name="lock" size="16px" class="q-mr-xs" />
                Servicio entregado — ya no se puede editar ni eliminar
              </q-card-section>

            </q-card>
          </div>

          <div v-if="!serviciosFiltrados.length" class="col-12 text-center q-pa-xl empty-box">
            <div class="text-subtitle1 text-grey-8">
              No hay servicios registrados en esta sección
            </div>

            <q-btn v-if="textoBusqueda || filtroEstado !== 'todos'" flat color="primary" label="Limpiar filtros" no-caps
              class="q-mt-sm" @click="limpiarFiltros" />
          </div>
        </div>

        <q-page-sticky position="bottom-right" :offset="[18, 18]">
          <q-btn fab icon="add" color="primary" @click="abrirModalNuevo" />
        </q-page-sticky>

      </q-page>
    </q-page-container>

    <q-dialog v-model="mostrarModalFormulario" persistent>
      <q-card class="modal">

        <q-card-section class="row items-center bg-primary text-white">
          <div class="text-h6 text-weight-bold">
            {{ modoEdicion ? 'Editar servicio' : 'Nuevo servicio' }}
          </div>

          <q-space />

          <q-btn flat round dense icon="close" @click="cerrarModalFormulario" />
        </q-card-section>

        <q-card-section class="scroll form-body">
          <q-form ref="formularioRef">
            <div class="row q-col-gutter-md">

              <div class="col-12 col-sm-6">
                <q-input v-model="formulario.cliente" outlined dense label="Nombre del cliente" />
              </div>

              <div class="col-6 col-sm-3">
                <q-select v-model="formulario.marca" :options="opcionesMarcasFiltradas" outlined dense use-input
                  fill-input hide-selected new-value-mode="add-unique" input-debounce="0" label="Marca"
                  @filter="filtrarMarcas" />
              </div>

              <div class="col-6 col-sm-3">
                <q-input v-model="formulario.modelo" outlined dense label="Modelo" />
              </div>

              <div class="col-12 col-sm-6">
                <q-select v-model="formulario.tipoReparacion" :options="opcionesTipoReparacion" emit-value map-options
                  outlined dense label="Tipo de reparación" />
              </div>

              <div class="col-12 col-sm-6">
                <q-select v-model="formulario.tecnico" :options="opcionesTecnicos" outlined dense label="Técnico" />
              </div>

              <div class="col-12 col-sm-6">
                <q-input :model-value="formatearFecha(formulario.fechaRecepcion)" outlined dense readonly disable
                  label="Fecha de recepción (automática)" hint="Se asigna sola al crear el servicio" />
              </div>

              <div class="col-6 col-sm-3">
                <q-input v-model.number="formulario.precio" type="number" outlined dense prefix="$" label="Precio" />
              </div>

              <div class="col-6 col-sm-3">
                <q-select v-model="formulario.metodoPago" :options="opcionesMetodoPago" outlined dense
                  label="Método de pago" />
              </div>

              <div class="col-12 col-sm-6">
                <q-select v-model="formulario.estadoPago" :options="opcionesEstadoPago" emit-value map-options outlined
                  dense label="Estado del pago" />
              </div>

              <!-- Caja de monto abonado: ahora sí aparece al elegir "Abono" -->
              <div v-if="formulario.estadoPago === 'abono'" class="col-12 col-sm-6">
                <q-input v-model.number="formulario.montoAbonado" type="number" outlined dense prefix="$"
                  label="Monto abonado" />
              </div>

              <div class="col-12" v-if="modoEdicion">
                <q-select v-model="formulario.estadoEquipo" :options="opcionesEstadoEquipo" emit-value map-options
                  outlined dense label="Estado del equipo" />
              </div>
              <div class="col-12" v-else>
                <q-banner dense class="estado-inicial-banner">
                  <template #avatar>
                    <q-icon name="inbox" color="primary" />
                  </template>
                  Estado inicial: <strong>Recibido</strong>
                </q-banner>
              </div>

              <div class="col-12">
                <q-input v-model="formulario.observaciones" type="textarea" autogrow outlined label="Observaciones" />
              </div>

            </div>
          </q-form>
        </q-card-section>

        <q-card-actions align="right" class="bg-grey-1">
          <q-btn flat label="Cancelar" color="grey-8" @click="cerrarModalFormulario" />

          <q-btn unelevated label="Guardar" color="primary" icon="save" @click="guardarServicio" />
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
          <q-btn flat label="Cancelar" @click="mostrarModalEliminar = false" />

          <q-btn unelevated color="negative" icon="delete" label="Sí, eliminar" @click="eliminarServicio" />
        </q-card-actions>

      </q-card>
    </q-dialog>

    <!-- AVISO (no usa alert() ni confirm() del navegador) -->
    <q-banner v-if="mensajeAviso" class="fixed-bottom text-white text-center text-weight-bold"
      :class="mensajeAvisoTipo === 'error' ? 'bg-negative' : 'bg-positive'" style="z-index:9999">
      {{ mensajeAviso }}
    </q-banner>

  </q-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const opcionesTipoReparacion = [
  { label: 'Cambio de pantalla', value: 'pantalla' },
  { label: 'Cambio de batería', value: 'bateria' },
  { label: 'Cambio de pin de carga', value: 'pin_carga' },
  { label: 'Liberación', value: 'liberacion' },
  { label: 'Mantenimiento de software', value: 'software' },
  { label: 'Cambio de flex', value: 'flex' },
  { label: 'Otros', value: 'otros' }
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
  { label: 'Pagado', value: 'pagado' },
  { label: 'Pendiente', value: 'pendiente' },
  { label: 'Abono', value: 'abono' }
]

const opcionesEstadoEquipo = [
  { label: 'Recibido', value: 'recibido' },
  { label: 'En reparación', value: 'en_reparacion' },
  { label: 'Listo para entregar', value: 'listo' },
  { label: 'Entregado', value: 'entregado' }
]

const opcionesFiltroEtapas = [
  { label: 'Todas las etapas', value: 'todos' },
  ...opcionesEstadoEquipo.map(x => ({
    label: x.label,
    value: x.value
  }))
]

const opcionesMarcas = [
  'Samsung',
  'Apple',
  'Xiaomi',
  'Motorola',
  'Huawei',
  'Oppo',
  'Realme',
  'ZTE',
  'Nokia',
  'Otro'
]

const opcionesMarcasFiltradas = ref(opcionesMarcas)

function filtrarMarcas(val, update) {
  update(() => {
    const texto = val.toLowerCase()
    opcionesMarcasFiltradas.value = opcionesMarcas.filter(
      m => m.toLowerCase().includes(texto)
    )
  })
}

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
  cliente: '',
  marca: '',
  modelo: '',
  tipoReparacion: null,
  tecnico: null,
  fechaRecepcion: '',
  precio: 0,
  metodoPago: null,
  estadoPago: null,
  montoAbonado: 0,
  estadoEquipo: 'recibido',
  calificacion: 0,
  observaciones: ''
})

const formulario = ref(formularioVacio())

const guardarDatos = () => {
  localStorage.setItem(
    'taller-don-efrain-servicios',
    JSON.stringify(servicios.value)
  )
}

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
    label: 'En taller',
    color: 'primary',
    valor: () =>
      contar('recibido') + contar('en_reparacion')
  },
  {
    label: 'Listos para entregar',
    color: 'positive',
    valor: () => contar('listo')
  },
  {
    label: 'Pagos pendientes',
    color: 'negative',
    valor: () =>
      servicios.value.filter(s => s.estadoPago === 'pendiente').length
  },
  {
    label: 'Total fiado',
    color: 'teal-8',
    valor: () => '$' + formatearNumero(totalFiado())
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
    day: '2-digit',
    month: 'short',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}

function obtenerFechaHoraActual() {
  const d = new Date()
  d.setMinutes(d.getMinutes() - d.getTimezoneOffset())
  return d.toISOString().slice(0, 16)
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
    recibido: 'blue-grey',
    en_reparacion: 'orange-9',
    listo: 'positive',
    entregado: 'grey-7'
  }[estado] || 'grey'
}

function abrirModalNuevo() {
  modoEdicion.value = false
  idEnEdicion.value = null
  formulario.value = formularioVacio()
  // La fecha se asigna sola, el usuario no la edita
  formulario.value.fechaRecepcion = obtenerFechaHoraActual()
  // El estado del equipo siempre arranca en "recibido"
  formulario.value.estadoEquipo = 'recibido'
  mostrarModalFormulario.value = true
}

function abrirModalEditar(servicio) {
  // Un servicio entregado no se puede editar (se controla también en la UI,
  // pero se valida aquí por seguridad)
  if (servicio.estadoEquipo === 'entregado') {
    return mostrarAviso(
      'Este servicio ya fue entregado y no se puede editar',
      'error'
    )
  }

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
  // Se permiten campos en blanco: no se valida el formulario antes de guardar
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
      id: idEnEdicion.value
    }

    mostrarAviso('Servicio actualizado correctamente')
  } else {
    servicios.value.unshift({
      ...formulario.value,
      id: Date.now().toString()
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
  // Un servicio entregado no se puede eliminar
  if (servicio.estadoEquipo === 'entregado') {
    return mostrarAviso(
      'Este servicio ya fue entregado y no se puede eliminar',
      'error'
    )
  }

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
  background: #f8fafc;
  color: #1e293b;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  font-size: 15px;
}

.header-taller {
  background: #095de4;
  padding: 10px 20%;
}

.header-content {
  padding: 10px 24px;
  gap: 24px;
}

.brand {
  display: flex;
  align-items: center;
  gap: 20px;
}

.logo {
  width: 90px;
  max-height: 70px;
  object-fit: contain;
}

.search {
  width: 320px;
  max-width: 40vw;
}

.page-content {
  min-height: 100vh;
  background: #f8fafc;
}

.stat-card,
.control-bar,
.device-card,
.empty-box {
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
}

.device-card {
  box-shadow: 0 1px 3px rgba(0, 0, 0, .05);
  overflow: hidden;
  font-size: 14.5px;
}

/* Letra más grande y legible dentro de las tarjetas de servicio */
.device-card .text-caption {
  font-size: 13.5px;
}

.device-card .text-subtitle1 {
  font-size: 17px;
}

.device-card .text-subtitle2 {
  font-size: 15px;
}

.device-card .marca-modelo {
  font-size: 15px;
}

.device-card .q-badge {
  font-size: 12.5px;
  padding: 4px 8px;
}

.stat-card .text-h5 {
  font-size: 22px;
}

.entregado-lock {
  font-size: 13.5px;
  color: #64748b;
  background: #f1f5f9;
}

.estado-inicial-banner {
  background: #eef2ff;
  border-radius: 8px;
  font-size: 14px;
}

.payment {
  padding: 6px;
  text-align: center;
  color: #fff;
  font-size: 13px;
  font-weight: bold;
}

.pending {
  background: #c10015;
}

.abonado {
  background: #f57c00;
}

.modal {
  width: 650px;
  max-width: 95vw;
  border-radius: 12px;
  overflow: hidden;
}

.form-body {
  max-height: 70vh;
}

.delete-modal {
  width: 400px;
  max-width: 90vw;
}

@media (max-width:700px) {
  .header-content {
    flex-wrap: wrap;
  }

  .brand {
    width: 100%;
    justify-content: space-between;
  }

  .search {
    width: 100%;
    max-width: none;
  }

  .logo {
    width: 70px;
  }
}
</style>