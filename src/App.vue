<template>
  <q-layout view="hHh lpR fFf" class="app-layout">

    <!-- =========================================================
         HEADER
    ========================================================== -->

    <q-header elevated class="header-taller">

      <q-toolbar class="header-content">

        <div class="brand">

          <div>

            <div class="text-h5 text-weight-bold">
              Efraín Tech Service
            </div>

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


    <!-- =========================================================
         CONTENIDO
    ========================================================== -->

    <q-page-container>

      <q-page class="page-content q-pa-md">


        <!-- =====================================================
             RESUMEN
        ====================================================== -->

        <div class="row q-col-gutter-md q-mb-md">

          <div v-for="item in resumen" :key="item.label" class="col-6 col-sm-3">

            <q-card flat class="stat-card">

              <q-card-section class="text-center q-pa-sm">

                <div class="text-caption text-grey-7">
                  {{ item.label }}
                </div>

                <div :class="`text-h5 text-weight-bold text-${item.color}`">
                  {{ item.valor() }}
                </div>

              </q-card-section>

            </q-card>

          </div>

        </div>


        <!-- =====================================================
             FILTROS
        ====================================================== -->

        <div class="control-bar row items-center justify-between q-pa-sm q-mb-md">

          <div class="text-subtitle1 text-weight-bold">
            Servicios Registrados
            ({{ serviciosFiltrados.length }})
          </div>

          <q-select v-model="filtroEstado" :options="opcionesFiltroEtapas" emit-value map-options dense outlined
            bg-color="white" label="Etapa de trabajo" style="min-width:220px" />

        </div>


        <!-- =====================================================
             TARJETAS DE SERVICIOS
        ====================================================== -->

        <div class="row q-col-gutter-md">

          <div v-for="servicio in serviciosFiltrados" :key="servicio.id" class="col-12 col-sm-6 col-md-4">

            <q-card flat class="device-card">


              <!-- =================================================
                   ESTADO DE PAGO
              ================================================== -->

              <div v-if="servicio.estadoPago === 'pendiente'" class="payment pending">
                PAGO PENDIENTE
              </div>


              <div v-else-if="servicio.estadoPago === 'abono'" class="payment abonado">

                ABONADO

                — Falta
                ${{ formatearNumero(calcularSaldo(servicio)) }}

              </div>


              <!-- =================================================
                   DATOS PRINCIPALES
              ================================================== -->

              <q-card-section>

                <div class="row items-center justify-between">

                  <div>

                    <div class="text-subtitle1 text-weight-bold">
                      {{ servicio.cliente || 'Sin nombre' }}
                    </div>

                    <div class="text-primary text-weight-medium marca-modelo">
                      {{ servicio.marca || 'Sin marca' }}
                      {{ servicio.modelo || 'Sin modelo' }}
                    </div>

                  </div>

                </div>


                <!-- =================================================
                     REPARACIONES
                ================================================== -->

                <div class="q-mt-sm reparaciones">

                  <div class="text-caption text-grey-7 q-mb-xs">
                    Reparaciones:
                  </div>

                  <div class="row q-gutter-xs">

                    <q-badge v-for="tipo in normalizarReparaciones(
                      servicio.tipoReparacion
                    )" :key="tipo" color="grey-3" text-color="grey-9">

                      {{ etiquetaReparacion(tipo) }}

                    </q-badge>

                  </div>

                </div>

              </q-card-section>


              <q-separator />


              <!-- =================================================
                   DETALLES
              ================================================== -->

              <q-card-section class="text-caption">


                <!-- ESTADO -->

                <div class="row justify-between q-mb-sm">

                  <span class="text-grey-7">
                    Etapa:
                  </span>

                  <q-badge :color="colorEstado(
                    servicio.estadoEquipo
                  )" class="text-weight-bold">

                    {{ etiquetaEstado(
                      servicio.estadoEquipo
                    ) }}

                  </q-badge>

                </div>


                <!-- TÉCNICO Y FECHA -->

                <div class="row justify-between text-grey-8">

                  <span>
                    Técnico:
                    {{ servicio.tecnico || '—' }}
                  </span>

                  <span>
                    {{ formatearFecha(
                      servicio.fechaRecepcion
                    ) }}
                  </span>

                </div>


                <!-- PRECIO -->

                <div class="row justify-between q-mt-sm text-subtitle2">

                  <span>

                    Precio:

                    <strong>
                      ${{ formatearNumero(
                        servicio.precio
                      ) }}
                    </strong>

                  </span>

                  <span class="text-caption text-grey-7">
                    {{ servicio.metodoPago || '—' }}
                  </span>

                </div>


                <!-- =================================================
                     INFORMACIÓN DE ABONOS
                ================================================== -->

                <div v-if="
                  servicio.estadoPago === 'abono' ||
                  Number(servicio.montoAbonado) > 0
                " class="payment-info q-mt-md">

                  <div class="row justify-between">

                    <span>
                      Total abonado:
                    </span>

                    <strong class="text-positive">
                      ${{ formatearNumero(
                        servicio.montoAbonado
                      ) }}
                    </strong>

                  </div>


                  <div class="row justify-between q-mt-xs">

                    <span>
                      Saldo pendiente:
                    </span>

                    <strong class="text-negative">
                      ${{ formatearNumero(
                        calcularSaldo(servicio)
                      ) }}
                    </strong>

                  </div>

                </div>


                <!-- OBSERVACIONES -->

                <div v-if="servicio.observaciones" class="text-grey-7 text-italic q-mt-sm">

                  Obs:
                  {{ servicio.observaciones }}

                </div>


                <!-- CALIFICACIÓN -->

                <div v-if="
                  servicio.estadoEquipo ===
                  'entregado'
                " class="row items-center justify-between q-mt-sm">

                  <span>
                    Calificación:
                  </span>

                  <q-rating v-model="servicio.calificacion" size="1.3em" color="amber" icon="star_border"
                    icon-selected="star" @update:model-value="
                      guardarCambiosDirectos(servicio)
                      " />

                </div>

              </q-card-section>


              <q-separator />


              <!-- =================================================
                   BOTONES
              ================================================== -->

              <q-card-actions v-if="
                servicio.estadoEquipo !==
                'entregado'
              " align="right">

                <q-btn flat dense icon="edit" color="primary" label="Editar" no-caps @click="
                  abrirModalEditar(servicio)
                  " />

                <q-btn flat dense icon="delete" color="negative" label="Eliminar" no-caps @click="
                  abrirConfirmarEliminar(servicio)
                  " />

              </q-card-actions>


              <!-- ENTREGADO -->

              <q-card-section v-else class="text-center entregado-lock">

                <q-icon name="lock" size="16px" class="q-mr-xs" />

                Servicio entregado —
                ya no se puede editar ni eliminar

              </q-card-section>

            </q-card>

          </div>


          <!-- =====================================================
               SIN SERVICIOS
          ====================================================== -->

          <div v-if="!serviciosFiltrados.length" class="col-12 text-center q-pa-xl empty-box">

            <div class="text-subtitle1 text-grey-8">
              No hay servicios registrados en esta sección
            </div>

            <q-btn v-if="
              textoBusqueda ||
              filtroEstado !== 'todos'
            " flat color="primary" label="Limpiar filtros" no-caps class="q-mt-sm" @click="limpiarFiltros" />

          </div>

        </div>


        <!-- =====================================================
             BOTÓN NUEVO
        ====================================================== -->

        <q-page-sticky position="bottom-right" :offset="[18, 18]">

          <q-btn fab icon="add" color="primary" @click="abrirModalNuevo" />

        </q-page-sticky>

      </q-page>

    </q-page-container>


    <!-- =========================================================
         MODAL NUEVO / EDITAR
    ========================================================== -->

    <q-dialog v-model="mostrarModalFormulario" persistent>

      <q-card class="modal">


        <!-- CABECERA -->

        <q-card-section class="row items-center bg-primary text-white">

          <div class="text-h6 text-weight-bold">

            {{
              modoEdicion
                ? 'Editar servicio'
                : 'Nuevo servicio'
            }}

          </div>

          <q-space />

          <q-btn flat round dense icon="close" @click="
            cerrarModalFormulario
          " />

        </q-card-section>


        <!-- CUERPO -->

        <q-card-section class="scroll form-body">

          <q-form ref="formularioRef">

            <div class="row q-col-gutter-md">


              <!-- =================================================
                   CLIENTE
              ================================================== -->

              <div class="col-12 col-sm-6">

                <q-input v-model="formulario.cliente" outlined dense label="Nombre del cliente" :rules="[
                  val =>
                    !!val?.trim() ||
                    'El nombre del cliente es obligatorio'
                ]" />

              </div>


              <!-- =================================================
                   MARCA
              ================================================== -->

              <div class="col-6 col-sm-3">

                <q-select v-model="formulario.marca" :options="opcionesMarcasFiltradas
                  " outlined dense use-input fill-input hide-selected new-value-mode="add-unique" input-debounce="0"
                  label="Marca" :rules="[
                    val =>
                      !!val ||
                      'La marca es obligatoria'
                  ]" @filter="
                    filtrarMarcas
                  " @update:model-value="
                    cambiarMarca
                  " />

              </div>


              <!-- =================================================
                   MODELO
              ================================================== -->

              <div class="col-6 col-sm-3">

                <q-select v-model="formulario.modelo" :options="opcionesModelosFiltrados
                  " outlined dense use-input fill-input hide-selected new-value-mode="add-unique" input-debounce="0"
                  label="Modelo" :disable="!formulario.marca" :hint="!formulario.marca
                      ? 'Primero selecciona una marca'
                      : 'Selecciona o escribe el modelo'
                    " :rules="[
                    val =>
                      !!val ||
                      'El modelo es obligatorio'
                  ]" />

              </div>


              <!-- =================================================
                   TIPO DE REPARACIÓN MÚLTIPLE
              ================================================== -->

              <div class="col-12">

                <q-select v-model="formulario.tipoReparacion
                  " :options="opcionesTipoReparacion
                    " emit-value map-options multiple use-chips outlined dense label="Tipos de reparación"
                  hint="Puedes seleccionar varias reparaciones" :rules="[
                    val =>
                      Array.isArray(val) &&
                      val.length > 0 ||
                      'Selecciona al menos una reparación'
                  ]">

                  <template #selected-item="scope">

                    <q-chip removable dense color="primary" text-color="white" @remove="
                      scope.removeAtIndex(
                        scope.index
                      )
                      ">

                      {{ scope.opt.label }}

                    </q-chip>

                  </template>

                </q-select>

              </div>


              <!-- =================================================
                   TÉCNICO
              ================================================== -->

              <div class="col-12 col-sm-6">

                <q-select v-model="formulario.tecnico" :options="opcionesTecnicos" outlined dense label="Técnico"
                  :rules="[
                    val =>
                      !!val ||
                      'Selecciona un técnico'
                  ]" />

              </div>


              <!-- =================================================
                   FECHA
              ================================================== -->

              <div class="col-12 col-sm-6">

                <q-input :model-value="formatearFecha(
                  formulario.fechaRecepcion
                )
                  " outlined dense readonly disable label="Fecha de recepción (automática)"
                  hint="Se asigna sola al crear el servicio" />

              </div>


              <!-- =================================================
                   PRECIO
              ================================================== -->

              <div class="col-6 col-sm-3">

                <q-input v-model.number="formulario.precio
                  " type="number" outlined dense prefix="$" label="Precio total" :rules="[
                    val =>
                      val !== null &&
                      val !== '' ||
                      'El precio es obligatorio',

                    val =>
                      Number(val) >= 0 ||
                      'El precio no puede ser negativo'
                  ]" />

              </div>


              <!-- =================================================
                   MÉTODO DE PAGO
              ================================================== -->

              <div class="col-6 col-sm-3">

                <q-select v-model="formulario.metodoPago
                  " :options="opcionesMetodoPago
                    " outlined dense label="Método de pago" :rules="[
                    val =>
                      !!val ||
                      'Selecciona un método de pago'
                  ]" />

              </div>


              <!-- =================================================
                   ESTADO DEL PAGO
              ================================================== -->

              <div class="col-12 col-sm-6">

                <q-select v-model="formulario.estadoPago
                  " :options="opcionesEstadoPago
                    " emit-value map-options outlined dense label="Estado del pago" :rules="[
                    val =>
                      !!val ||
                      'Selecciona el estado del pago'
                  ]" />

              </div>


              <!-- =================================================
                   ABONO NUEVO
              ================================================== -->

              <div v-if="
                formulario.estadoPago ===
                'abono'
              " class="col-12 col-sm-6">

                <q-input v-model.number="formulario.abonoActual
                  " type="number" outlined dense prefix="$" label="Abono de esta vez" hint="
                    Este valor se sumará al total abonado
                  " :rules="[
                    val =>
                      val !== null &&
                      val !== '' ||
                      'Ingresa el abono',

                    val =>
                      Number(val) > 0 ||
                      'El abono debe ser mayor que cero',

                    val =>
                      Number(val) <=
                      calcularMaximoAbono() ||
                      `Máximo permitido: $${formatearNumero(
                        calcularMaximoAbono()
                      )}`
                  ]" />

              </div>


              <!-- =================================================
                   RESUMEN DE ABONOS EN EDICIÓN
              ================================================== -->

              <div v-if="
                modoEdicion &&
                Number(
                  formulario.montoAbonado
                ) > 0
              " class="col-12">

                <q-banner dense class="resumen-abono">

                  <template #avatar>

                    <q-icon name="payments" color="positive" />

                  </template>

                  <div>

                    <strong>
                      Total abonado anteriormente:
                    </strong>

                    ${{ formatearNumero(
                      formulario.montoAbonado
                    ) }}

                    <br>

                    <strong>
                      Saldo actual:
                    </strong>

                    ${{ formatearNumero(
                      calcularSaldo(formulario)
                    ) }}

                  </div>

                </q-banner>

              </div>


              <!-- =================================================
                   ESTADO DEL EQUIPO
              ================================================== -->

              <div v-if="modoEdicion" class="col-12">

                <q-select v-model="formulario.estadoEquipo
                  " :options="opcionesEstadoEquipo
                    " emit-value map-options outlined dense label="Estado del equipo" :rules="[
                    val =>
                      !!val ||
                      'Selecciona el estado del equipo'
                  ]" />

              </div>


              <!-- ESTADO INICIAL -->

              <div v-else class="col-12">

                <q-banner dense class="estado-inicial-banner">

                  <template #avatar>

                    <q-icon name="inbox" color="primary" />

                  </template>

                  Estado inicial:
                  <strong>Recibido</strong>

                </q-banner>

              </div>


              <!-- =================================================
                   OBSERVACIONES
              ================================================== -->

              <div class="col-12">

                <q-input v-model="formulario.observaciones
                  " type="textarea" autogrow outlined label="Observaciones" />

              </div>

            </div>

          </q-form>

        </q-card-section>


        <!-- BOTONES -->

        <q-card-actions align="right" class="bg-grey-1">

          <q-btn flat label="Cancelar" color="grey-8" @click="
            cerrarModalFormulario
          " />

          <q-btn unelevated label="Guardar" color="primary" icon="save" @click="
            guardarServicio
          " />

        </q-card-actions>

      </q-card>

    </q-dialog>


    <!-- =========================================================
         MODAL ELIMINAR
    ========================================================== -->

    <q-dialog v-model="mostrarModalEliminar">

      <q-card class="delete-modal">

        <q-card-section class="bg-negative text-white">

          <div class="text-h6 text-weight-bold">
            ¿Eliminar servicio?
          </div>

        </q-card-section>


        <q-card-section>

          Vas a eliminar el registro de

          <strong>
            {{ servicioAEliminar?.cliente }}
          </strong>

          ({{ servicioAEliminar?.marca }}
          {{ servicioAEliminar?.modelo }}).

          <br>
          <br>

          Esta acción no se puede deshacer.

        </q-card-section>


        <q-card-actions align="right">

          <q-btn flat label="Cancelar" @click="
            mostrarModalEliminar = false
            " />

          <q-btn unelevated color="negative" icon="delete" label="Sí, eliminar" @click="
            eliminarServicio
          " />

        </q-card-actions>

      </q-card>

    </q-dialog>


    <!-- =========================================================
         AVISO
    ========================================================== -->

    <q-banner v-if="mensajeAviso" class="fixed-bottom text-white text-center text-weight-bold" :class="mensajeAvisoTipo === 'error'
        ? 'bg-negative'
        : 'bg-positive'
      " style="z-index:9999">

      {{ mensajeAviso }}

    </q-banner>

  </q-layout>
</template>


<script setup>

import {
  ref,
  computed
} from 'vue'


/* =========================================================
   TIPOS DE REPARACIÓN
========================================================= */

const opcionesTipoReparacion = [

  {
    label: 'Cambio de pantalla',
    value: 'pantalla'
  },

  {
    label: 'Cambio de batería',
    value: 'bateria'
  },

  {
    label: 'Cambio de pin de carga',
    value: 'pin_carga'
  },

  {
    label: 'Liberación',
    value: 'liberacion'
  },

  {
    label: 'Mantenimiento de software',
    value: 'software'
  },

  {
    label: 'Cambio de flex',
    value: 'flex'
  },

  {
    label: 'Otros',
    value: 'otros'
  }

]


/* =========================================================
   TÉCNICOS
========================================================= */

const opcionesTecnicos = [

  'Efraín Sarmiento Vesga',

  'Helver Remolina Rondon',

  'Sergio Aguirre Baez'

]


/* =========================================================
   MÉTODOS DE PAGO
========================================================= */

const opcionesMetodoPago = [

  'Efectivo',

  'Transferencia',

  'Tarjeta'

]


/* =========================================================
   ESTADOS DE PAGO
========================================================= */

const opcionesEstadoPago = [

  {
    label: 'Pagado',
    value: 'pagado'
  },

  {
    label: 'Pendiente',
    value: 'pendiente'
  },

  {
    label: 'Abono',
    value: 'abono'
  }

]


/* =========================================================
   ESTADOS DEL EQUIPO
========================================================= */

const opcionesEstadoEquipo = [

  {
    label: 'Recibido',
    value: 'recibido'
  },

  {
    label: 'En reparación',
    value: 'en_reparacion'
  },

  {
    label: 'Listo para entregar',
    value: 'listo'
  },

  {
    label: 'Entregado',
    value: 'entregado'
  }

]


/* =========================================================
   FILTROS
========================================================= */

const opcionesFiltroEtapas = [

  {
    label: 'Todas las etapas',
    value: 'todos'
  },

  ...opcionesEstadoEquipo.map(
    x => ({
      label: x.label,
      value: x.value
    })
  )

]


/* =========================================================
   MARCAS Y MODELOS
========================================================= */

const modelosPorMarca = {

  Samsung: [

    'Galaxy A03',
    'Galaxy A04',
    'Galaxy A05',
    'Galaxy A06',

    'Galaxy A13',
    'Galaxy A14',
    'Galaxy A15',
    'Galaxy A16',

    'Galaxy A22',
    'Galaxy A23',
    'Galaxy A24',
    'Galaxy A25',

    'Galaxy A32',
    'Galaxy A33',
    'Galaxy A34',
    'Galaxy A35',

    'Galaxy A52',
    'Galaxy A53',
    'Galaxy A54',
    'Galaxy A55',

    'Galaxy A72',
    'Galaxy A73',

    'Galaxy S20',
    'Galaxy S21',
    'Galaxy S21 FE',

    'Galaxy S22',
    'Galaxy S22+',
    'Galaxy S22 Ultra',

    'Galaxy S23',
    'Galaxy S23+',
    'Galaxy S23 Ultra',

    'Galaxy S24',
    'Galaxy S24+',
    'Galaxy S24 Ultra',

    'Galaxy S25',
    'Galaxy S25+',
    'Galaxy S25 Ultra',

    'Galaxy Note 10',
    'Galaxy Note 20',
    'Galaxy Note 20 Ultra',

    'Galaxy Z Flip 3',
    'Galaxy Z Flip 4',
    'Galaxy Z Flip 5',
    'Galaxy Z Flip 6',

    'Galaxy Z Fold 3',
    'Galaxy Z Fold 4',
    'Galaxy Z Fold 5',
    'Galaxy Z Fold 6'

  ],


  Apple: [

    'iPhone 8',
    'iPhone 8 Plus',

    'iPhone X',
    'iPhone XR',
    'iPhone XS',
    'iPhone XS Max',

    'iPhone 11',
    'iPhone 11 Pro',
    'iPhone 11 Pro Max',

    'iPhone SE 2020',

    'iPhone 12',
    'iPhone 12 mini',
    'iPhone 12 Pro',
    'iPhone 12 Pro Max',

    'iPhone 13',
    'iPhone 13 mini',
    'iPhone 13 Pro',
    'iPhone 13 Pro Max',

    'iPhone SE 2022',

    'iPhone 14',
    'iPhone 14 Plus',
    'iPhone 14 Pro',
    'iPhone 14 Pro Max',

    'iPhone 15',
    'iPhone 15 Plus',
    'iPhone 15 Pro',
    'iPhone 15 Pro Max',

    'iPhone 16',
    'iPhone 16 Plus',
    'iPhone 16 Pro',
    'iPhone 16 Pro Max',
    'iPhone 16e',

    'iPhone 17',
    'iPhone 17 Air',
    'iPhone 17 Pro',
    'iPhone 17 Pro Max'

  ],


  Xiaomi: [

    'Redmi 9',
    'Redmi 9A',
    'Redmi 9C',

    'Redmi 10',
    'Redmi 10C',

    'Redmi 12',
    'Redmi 13',

    'Redmi Note 9',
    'Redmi Note 10',
    'Redmi Note 10 Pro',

    'Redmi Note 11',
    'Redmi Note 11 Pro',

    'Redmi Note 12',
    'Redmi Note 12 Pro',

    'Redmi Note 13',
    'Redmi Note 13 Pro',

    'Redmi Note 14',
    'Redmi Note 14 Pro',

    'Poco X3',
    'Poco X3 Pro',

    'Poco X4 Pro',

    'Poco X5',
    'Poco X5 Pro',

    'Poco X6',
    'Poco X6 Pro',

    'Poco X7',

    'Poco M3',
    'Poco M4 Pro',
    'Poco M5',
    'Poco M6',

    'Mi 10',
    'Mi 11',
    'Mi 12',
    'Mi 13',
    'Mi 14'

  ],


  Motorola: [

    'Moto E6',
    'Moto E7',

    'Moto E13',
    'Moto E14',

    'Moto E20',
    'Moto E22',
    'Moto E32',
    'Moto E40',

    'Moto G8',
    'Moto G9',
    'Moto G10',

    'Moto G20',
    'Moto G22',
    'Moto G30',
    'Moto G31',
    'Moto G32',

    'Moto G41',
    'Moto G42',

    'Moto G51',
    'Moto G52',
    'Moto G53',
    'Moto G54',

    'Moto G60',
    'Moto G62',

    'Moto G72',
    'Moto G73',

    'Moto G82',
    'Moto G84',
    'Moto G85',

    'Moto G100',
    'Moto G200',

    'Motorola Edge 20',
    'Motorola Edge 30',
    'Motorola Edge 40',
    'Motorola Edge 50',

    'Motorola Razr 40',
    'Motorola Razr 50'

  ],


  Huawei: [

    'P20',
    'P20 Lite',

    'P30',
    'P30 Lite',
    'P30 Pro',

    'P40',
    'P40 Lite',
    'P40 Pro',

    'P50',
    'P50 Pro',

    'P60',
    'P60 Pro',

    'Nova 5T',
    'Nova 7i',
    'Nova 8i',
    'Nova 9',
    'Nova 10',
    'Nova 11',

    'Y5',
    'Y6',
    'Y7',
    'Y8',
    'Y9',
    'Y9 Prime',
    'Y9a',
    'Y70'

  ],


  Oppo: [

    'A12',
    'A15',
    'A16',
    'A17',
    'A18',

    'A31',
    'A53',
    'A54',
    'A57',
    'A58',

    'A74',
    'A76',
    'A77',
    'A78',
    'A79',

    'A96',

    'Reno 4',
    'Reno 5',
    'Reno 6',
    'Reno 7',
    'Reno 8',
    'Reno 10',
    'Reno 11',
    'Reno 12'

  ],


  Realme: [

    'C11',
    'C12',
    'C15',

    'C20',
    'C21',
    'C25',

    'C30',
    'C31',
    'C33',
    'C35',

    'C51',
    'C53',
    'C55',

    'C61',
    'C63',
    'C67',
    'C75',

    'Narzo 30',
    'Narzo 50',
    'Narzo 60',

    'GT',
    'GT Neo 2',
    'GT Neo 3',
    'GT 5G'

  ],


  ZTE: [

    'Blade A3',
    'Blade A31',
    'Blade A51',
    'Blade A52',
    'Blade A53',
    'Blade A54',

    'Blade A71',
    'Blade A72',
    'Blade A73',

    'Blade V2020',
    'Blade V30',
    'Blade V40',
    'Blade V50',

    'Axon 20',
    'Axon 30',
    'Axon 40',
    'Axon 50'

  ],


  Nokia: [

    'Nokia 1',
    'Nokia 2',
    'Nokia 2.2',
    'Nokia 2.3',

    'Nokia 3',
    'Nokia 3.1',
    'Nokia 3.2',

    'Nokia 4.2',

    'Nokia 5',
    'Nokia 5.1',
    'Nokia 5.3',

    'Nokia 6',
    'Nokia 6.1',
    'Nokia 6.2',

    'Nokia 7.2',
    'Nokia 8',

    'Nokia C10',
    'Nokia C20',
    'Nokia C21',
    'Nokia C22',
    'Nokia C32',

    'Nokia G10',
    'Nokia G11',
    'Nokia G20',
    'Nokia G21',
    'Nokia G22',
    'Nokia G42',

    'Nokia X10',
    'Nokia X20',
    'Nokia X30'

  ],


  Otro: []

}


/* =========================================================
   MARCAS
========================================================= */

const opcionesMarcas =
  Object.keys(modelosPorMarca)

const opcionesMarcasFiltradas =
  ref(opcionesMarcas)


/* =========================================================
   MODELOS
========================================================= */

const opcionesModelosFiltrados =
  ref([])


/* =========================================================
   FILTRAR MARCAS
========================================================= */

function filtrarMarcas(
  val,
  update
) {

  update(() => {

    const texto =
      String(val || '')
        .toLowerCase()

    opcionesMarcasFiltradas.value =
      opcionesMarcas.filter(
        marca =>
          marca
            .toLowerCase()
            .includes(texto)
      )

  })

}


/* =========================================================
   CAMBIAR MARCA
========================================================= */

function cambiarMarca(nuevaMarca) {

  opcionesModelosFiltrados.value =
    modelosPorMarca[nuevaMarca] || []

  /*
   * Si el modelo actual no pertenece
   * a la nueva marca, se limpia.
   */

  const modeloActual =
    formulario.value.modelo

  if (
    modeloActual &&
    !opcionesModelosFiltrados.value.includes(
      modeloActual
    )
  ) {

    formulario.value.modelo =
      ''

  }

}


/* =========================================================
   SERVICIOS
========================================================= */

const servicios = ref(

  JSON.parse(

    localStorage.getItem(
      'taller-don-efrain-servicios'
    ) || '[]'

  ).map(servicio => ({

    ...servicio,

    /*
     * Compatibilidad con servicios
     * antiguos que tenían un solo
     * tipo de reparación.
     */

    tipoReparacion:
      normalizarReparaciones(
        servicio.tipoReparacion
      ),

    montoAbonado:
      Number(
        servicio.montoAbonado
      ) || 0,

    calificacion:
      Number(
        servicio.calificacion
      ) || 0

  }))

)


/* =========================================================
   BÚSQUEDA
========================================================= */

const textoBusqueda =
  ref('')


const filtroEstado =
  ref('todos')


/* =========================================================
   MODALES
========================================================= */

const mostrarModalFormulario =
  ref(false)

const mostrarModalEliminar =
  ref(false)

const modoEdicion =
  ref(false)

const idEnEdicion =
  ref(null)

const servicioAEliminar =
  ref(null)

const formularioRef =
  ref(null)


/* =========================================================
   MENSAJES
========================================================= */

const mensajeAviso =
  ref('')

const mensajeAvisoTipo =
  ref('exito')

let temporizadorAviso


/* =========================================================
   FORMULARIO
========================================================= */

const formularioVacio = () => ({

  cliente: '',

  marca: '',

  modelo: '',

  tipoReparacion: [],

  tecnico: null,

  fechaRecepcion: '',

  precio: 0,

  metodoPago: null,

  estadoPago: null,

  /*
   * Total acumulado de abonos
   */

  montoAbonado: 0,

  /*
   * Abono que se está realizando
   * en esta operación.
   */

  abonoActual: 0,

  estadoEquipo: 'recibido',

  calificacion: 0,

  observaciones: ''

})


const formulario =
  ref(formularioVacio())


/* =========================================================
   NORMALIZAR REPARACIONES
========================================================= */

function normalizarReparaciones(valor) {

  if (Array.isArray(valor)) {

    return valor

  }

  if (valor) {

    return [valor]

  }

  return []

}


/* =========================================================
   GUARDAR DATOS
========================================================= */

const guardarDatos = () => {

  /*
   * No guardamos abonoActual porque
   * solo representa el abono de la
   * operación actual.
   */

  const datos =
    servicios.value.map(
      servicio => {

        const copia = {
          ...servicio
        }

        delete copia.abonoActual

        return copia

      }
    )


  localStorage.setItem(

    'taller-don-efrain-servicios',

    JSON.stringify(datos)

  )

}


/* =========================================================
   SERVICIOS FILTRADOS
========================================================= */

const serviciosFiltrados =
  computed(() => {

    const texto =
      textoBusqueda.value
        .trim()
        .toLowerCase()


    return servicios.value.filter(
      servicio =>

        (
          filtroEstado.value ===
          'todos' ||

          servicio.estadoEquipo ===
          filtroEstado.value
        )

        &&

        (
          !texto ||

          [

            servicio.cliente,

            servicio.marca,

            servicio.modelo

          ].some(
            valor =>

              String(valor || '')
                .toLowerCase()
                .includes(texto)

          )

        )

    )

  })


/* =========================================================
   RESUMEN
========================================================= */

const resumen = [

  {
    label: 'En taller',

    color: 'primary',

    valor: () =>

      contar('recibido') +
      contar('en_reparacion')

  },


  {
    label: 'Listos para entregar',

    color: 'positive',

    valor: () =>
      contar('listo')

  },


  {
    label: 'Pagos pendientes',

    color: 'negative',

    valor: () =>

      servicios.value.filter(
        s =>
          s.estadoPago ===
          'pendiente'
      ).length

  },


  {
    label: 'Total fiado',

    color: 'teal-8',

    valor: () =>

      '$' +
      formatearNumero(
        totalFiado()
      )

  }

]


/* =========================================================
   CONTAR
========================================================= */

function contar(estado) {

  return servicios.value.filter(
    servicio =>
      servicio.estadoEquipo ===
      estado
  ).length

}


/* =========================================================
   TOTAL FIADO
========================================================= */

function totalFiado() {

  return servicios.value

    .filter(
      servicio =>
        servicio.estadoPago ===
        'abono'
    )

    .reduce(
      (total, servicio) =>
        total +
        calcularSaldo(servicio),

      0
    )

}


/* =========================================================
   CALCULAR SALDO
========================================================= */

function calcularSaldo(servicio) {

  return Math.max(

    (
      Number(
        servicio.precio
      ) || 0
    )

    -

    (
      Number(
        servicio.montoAbonado
      ) || 0
    ),

    0

  )

}


/* =========================================================
   MÁXIMO ABONO
========================================================= */

function calcularMaximoAbono() {

  const precio =
    Number(
      formulario.value.precio
    ) || 0


  const acumulado =
    Number(
      formulario.value.montoAbonado
    ) || 0


  return Math.max(
    precio - acumulado,
    0
  )

}


/* =========================================================
   FORMATEAR NÚMERO
========================================================= */

function formatearNumero(valor) {

  return (
    Number(valor) || 0
  ).toLocaleString('es-CO')

}


/* =========================================================
   FORMATEAR FECHA
========================================================= */

function formatearFecha(fecha) {

  if (!fecha)
    return 'Sin fecha'


  const f =
    new Date(fecha)


  if (isNaN(f))
    return 'Fecha inválida'


  return f.toLocaleString(
    'es-CO',
    {

      day: '2-digit',

      month: 'short',

      year: 'numeric',

      hour: '2-digit',

      minute: '2-digit'

    }
  )

}


/* =========================================================
   FECHA ACTUAL
========================================================= */

function obtenerFechaHoraActual() {

  const d =
    new Date()


  d.setMinutes(

    d.getMinutes() -
    d.getTimezoneOffset()

  )


  return d
    .toISOString()
    .slice(0, 16)

}


/* =========================================================
   ETIQUETA REPARACIÓN
========================================================= */

function etiquetaReparacion(valor) {

  return opcionesTipoReparacion.find(

    opcion =>
      opcion.value === valor

  )?.label ||

    valor ||

    'Sin especificar'

}


/* =========================================================
   ETIQUETA ESTADO
========================================================= */

function etiquetaEstado(estado) {

  return opcionesEstadoEquipo.find(

    opcion =>
      opcion.value === estado

  )?.label ||

    'Sin estado'

}


/* =========================================================
   COLOR ESTADO
========================================================= */

function colorEstado(estado) {

  return {

    recibido:
      'blue-grey',

    en_reparacion:
      'orange-9',

    listo:
      'positive',

    entregado:
      'grey-7'

  }[estado] ||

    'grey'

}


/* =========================================================
   NUEVO SERVICIO
========================================================= */

function abrirModalNuevo() {

  modoEdicion.value =
    false

  idEnEdicion.value =
    null

  formulario.value =
    formularioVacio()


  formulario.value.fechaRecepcion =
    obtenerFechaHoraActual()


  formulario.value.estadoEquipo =
    'recibido'


  opcionesModelosFiltrados.value =
    []


  opcionesMarcasFiltradas.value =
    opcionesMarcas


  mostrarModalFormulario.value =
    true

}


/* =========================================================
   EDITAR SERVICIO
========================================================= */

function abrirModalEditar(servicio) {

  if (
    servicio.estadoEquipo ===
    'entregado'
  ) {

    return mostrarAviso(

      'Este servicio ya fue entregado y no se puede editar',

      'error'

    )

  }


  modoEdicion.value =
    true


  idEnEdicion.value =
    servicio.id


  formulario.value = {

    ...servicio,

    tipoReparacion:
      normalizarReparaciones(
        servicio.tipoReparacion
      ),

    montoAbonado:
      Number(
        servicio.montoAbonado
      ) || 0,

    /*
     * Al abrir edición empezamos
     * con un nuevo abono en cero.
     */

    abonoActual: 0

  }


  opcionesModelosFiltrados.value =
    modelosPorMarca[
    servicio.marca
    ] || []


  opcionesMarcasFiltradas.value =
    opcionesMarcas


  mostrarModalFormulario.value =
    true

}


/* =========================================================
   CERRAR MODAL
========================================================= */

function cerrarModalFormulario() {

  mostrarModalFormulario.value =
    false

  formulario.value =
    formularioVacio()

  modoEdicion.value =
    false

  idEnEdicion.value =
    null

  opcionesModelosFiltrados.value =
    []

}


/* =========================================================
   GUARDAR SERVICIO
========================================================= */

async function guardarServicio() {

  /*
   * VALIDACIONES DE QUASAR
   */

  const valido =
    await formularioRef.value?.validate()


  if (!valido) {

    mostrarAviso(

      'Por favor completa los campos obligatorios',

      'error'

    )

    return

  }


  const precio =
    Number(
      formulario.value.precio
    ) || 0


  let totalAbonado =
    Number(
      formulario.value.montoAbonado
    ) || 0


  const abonoActual =
    Number(
      formulario.value.abonoActual
    ) || 0


  /* =======================================================
     NUEVO ABONO
  ======================================================== */

  if (
    formulario.value.estadoPago ===
    'abono'
  ) {

    const saldoActual =
      Math.max(
        precio - totalAbonado,
        0
      )


    if (
      abonoActual <= 0
    ) {

      mostrarAviso(

        'Ingresa el valor del abono',

        'error'

      )

      return

    }


    if (
      abonoActual > saldoActual
    ) {

      mostrarAviso(

        'El abono no puede superar el saldo pendiente',

        'error'

      )

      return

    }


    /*
     * AQUÍ SE SUMA EL NUEVO ABONO
     * AL TOTAL ANTERIOR.
     */

    totalAbonado +=
      abonoActual

  }


  /* =======================================================
     PAGADO
  ======================================================== */

  if (
    formulario.value.estadoPago ===
    'pagado'
  ) {

    totalAbonado =
      precio

  }


  /* =======================================================
     PENDIENTE
  ======================================================== */

  if (
    formulario.value.estadoPago ===
    'pendiente'
  ) {

    totalAbonado =
      0

  }


  /*
   * Si el total abonado llegó al
   * precio, automáticamente queda
   * pagado.
   */

  if (
    totalAbonado >= precio &&
    precio > 0
  ) {

    totalAbonado =
      precio

    formulario.value.estadoPago =
      'pagado'

  }


  /* =======================================================
     OBJETO FINAL
  ======================================================== */

  const servicioFinal = {

    ...formulario.value,

    precio,

    montoAbonado:
      totalAbonado,

    tipoReparacion:
      normalizarReparaciones(
        formulario.value.tipoReparacion
      )

  }


  /*
   * No guardamos el abono temporal.
   */

  delete servicioFinal.abonoActual


  /* =======================================================
     EDITAR
  ======================================================== */

  if (modoEdicion.value) {

    const indice =
      servicios.value.findIndex(

        servicio =>
          servicio.id ===
          idEnEdicion.value

      )


    if (indice === -1) {

      mostrarAviso(

        'Servicio no encontrado',

        'error'

      )

      return

    }


    servicios.value[indice] = {

      ...servicioFinal,

      id:
        idEnEdicion.value

    }


    if (abonoActual > 0) {

      mostrarAviso(

        `Abono de $${formatearNumero(
          abonoActual
        )} registrado correctamente`

      )

    } else {

      mostrarAviso(
        'Servicio actualizado correctamente'
      )

    }

  }


  /* =======================================================
     NUEVO
  ======================================================== */

  else {

    servicios.value.unshift({

      ...servicioFinal,

      id:
        Date.now().toString()

    })


    if (abonoActual > 0) {

      mostrarAviso(

        `Servicio registrado con abono de $${formatearNumero(
          abonoActual
        )}`

      )

    } else {

      mostrarAviso(
        'Servicio registrado correctamente'
      )

    }

  }


  guardarDatos()

  cerrarModalFormulario()

}


/* =========================================================
   GUARDAR CALIFICACIÓN
========================================================= */

function guardarCambiosDirectos(servicio) {

  guardarDatos()

  mostrarAviso(
    'Calificación guardada'
  )

}


/* =========================================================
   CONFIRMAR ELIMINAR
========================================================= */

function abrirConfirmarEliminar(servicio) {

  if (
    servicio.estadoEquipo ===
    'entregado'
  ) {

    return mostrarAviso(

      'Este servicio ya fue entregado y no se puede eliminar',

      'error'

    )

  }


  servicioAEliminar.value =
    servicio


  mostrarModalEliminar.value =
    true

}


/* =========================================================
   ELIMINAR
========================================================= */

function eliminarServicio() {

  if (
    !servicioAEliminar.value
  ) {

    return

  }


  servicios.value =
    servicios.value.filter(

      servicio =>
        servicio.id !==
        servicioAEliminar.value.id

    )


  guardarDatos()


  mostrarModalEliminar.value =
    false


  servicioAEliminar.value =
    null


  mostrarAviso(
    'Servicio eliminado'
  )

}


/* =========================================================
   LIMPIAR FILTROS
========================================================= */

function limpiarFiltros() {

  textoBusqueda.value =
    ''

  filtroEstado.value =
    'todos'

}


/* =========================================================
   AVISOS
========================================================= */

function mostrarAviso(
  texto,
  tipo = 'exito'
) {

  mensajeAviso.value =
    texto

  mensajeAvisoTipo.value =
    tipo


  clearTimeout(
    temporizadorAviso
  )


  temporizadorAviso =
    setTimeout(() => {

      mensajeAviso.value =
        ''

    }, 2500)

}

</script>


<style scoped>
/* =========================================================
   GENERAL
========================================================= */

.app-layout {

  background: #f8fafc;

  color: #1e293b;

  font-family:
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    "Segoe UI",
    Roboto,
    sans-serif;

  font-size: 15px;

}


/* =========================================================
   HEADER
========================================================= */

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


/* =========================================================
   PÁGINA
========================================================= */

.page-content {

  min-height: 100vh;

  background: #f8fafc;

}


/* =========================================================
   TARJETAS
========================================================= */

.stat-card,
.control-bar,
.device-card,
.empty-box {

  background: #fff;

  border: 1px solid #e2e8f0;

  border-radius: 12px;

}


.device-card {

  box-shadow:
    0 1px 3px rgba(0, 0, 0, .05);

  overflow: hidden;

  font-size: 14.5px;

}


/* =========================================================
   TEXTOS
========================================================= */

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


/* =========================================================
   REPARACIONES
========================================================= */

.reparaciones {

  border-top: 1px solid #f1f5f9;

  padding-top: 8px;

}


/* =========================================================
   INFORMACIÓN DE ABONOS
========================================================= */

.payment-info {

  background: #f8fafc;

  border: 1px solid #e2e8f0;

  border-radius: 8px;

  padding: 9px 10px;

  font-size: 13.5px;

}


/* =========================================================
   BARRA DE ABONOS EN FORMULARIO
========================================================= */

.resumen-abono {

  background: #ecfdf5;

  border: 1px solid #bbf7d0;

  border-radius: 8px;

  font-size: 14px;

}


/* =========================================================
   SERVICIO ENTREGADO
========================================================= */

.entregado-lock {

  font-size: 13.5px;

  color: #64748b;

  background: #f1f5f9;

}


/* =========================================================
   ESTADO INICIAL
========================================================= */

.estado-inicial-banner {

  background: #eef2ff;

  border-radius: 8px;

  font-size: 14px;

}


/* =========================================================
   PAGOS
========================================================= */

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


/* =========================================================
   MODAL
========================================================= */

.modal {

  width: 650px;

  max-width: 95vw;

  border-radius: 12px;

  overflow: hidden;

}


.form-body {

  max-height: 70vh;

}


/* =========================================================
   ELIMINAR
========================================================= */

.delete-modal {

  width: 400px;

  max-width: 90vw;

}


/* =========================================================
   MÓVIL
========================================================= */

@media (max-width: 700px) {

  .header-taller {

    padding: 10px;

  }


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
