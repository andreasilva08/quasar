<template>
  <q-layout view="lHh Lpr lFf" class="bg-grey-2">
    <!-- Encabezado -->
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
       <q-avatar icon="build" color="white" text-color="primary" />
        <q-toolbar-title class="text-weight-bold" style="font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif; font-size: 25px;">
          Taller Don Efraín
        </q-toolbar-title>
        <q-btn
          flat
          round
          dense
          icon="add"
          color="white"
          @click="abrirModalNuevo"
        >
          <q-tooltip>Nuevo Servicio</q-tooltip>
        </q-btn>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="q-pa-md">
        <!-- Dashboard Rápido -->
        <div class="row q-col-gutter-sm q-mb-md">
          <div class="col-6 col-sm-3">
            <q-card flat bordered class="bg-white text-center q-pa-xs">
              <div class="text-caption text-grey-7">Total Servicios</div>
              <div class="text-h6 text-weight-bolder text-primary">{{ servicios.length }}</div>
            </q-card>
          </div>
          <div class="col-6 col-sm-3">
            <q-card flat bordered class="bg-white text-center q-pa-xs">
              <div class="text-caption text-grey-7">En Taller</div>
              <div class="text-h6 text-weight-bolder text-warning">{{ contarPorEstado('en_taller') }}</div>
            </q-card>
          </div>
          <div class="col-6 col-sm-3">
            <q-card flat bordered class="bg-white text-center q-pa-xs">
              <div class="text-caption text-grey-7">Por Cobrar/Abono</div>
              <div class="text-h6 text-weight-bolder text-negative">{{ contarPagosPendientes() }}</div>
            </q-card>
          </div>
          <div class="col-6 col-sm-3">
            <q-card flat bordered class="bg-white text-center q-pa-xs">
              <div class="text-caption text-grey-7">Listos p/ Entregar</div>
              <div class="text-h6 text-weight-bolder text-positive">{{ contarPorEstado('listo') }}</div>
            </q-card>
          </div>
        </div>

        <!-- Búsqueda y Filtros -->
        <div class="row q-col-gutter-sm q-mb-md items-center">
          <div class="col-12 col-sm-6">
            <q-input
              v-model="filtroTexto"
              dense
              outlined
              placeholder="Buscar por cliente, equipo o técnico..."
              clearable
              bg-color="white"
            >
              <template v-slot:append>
                <q-icon name="search" />
              </template>
            </q-input>
          </div>
          <div class="col-12 col-sm-6">
            <q-select
              v-model="filtroEstado"
              :options="opcionesFiltroEstado"
              dense
              outlined
              emit-value
              map-options
              bg-color="white"
              label="Filtrar por estado del equipo"
            />
          </div>
        </div>

        <!-- Botón de Registro Principal -->
        <div class="row q-mb-md">
          <q-btn
            color="primary"
            icon="add_circle"
            label="Registrar Nuevo Servicio"
            class="full-width text-weight-bold"
            size="lg"
            unelevated
            @click="abrirModalNuevo"
          />
        </div>

        <!-- Mensaje cuando no existen registros -->
        <div v-if="obtenerServiciosFiltrados().length === 0" class="text-center q-pa-xl text-grey-6">
          <q-icon name="handyman" size="64px" color="grey-4" />
          <div class="text-h6 q-mt-md">No hay servicios registrados o no coinciden con la búsqueda</div>
        </div>

        <!-- Listado de Tarjetas -->
        <div class="row q-col-gutter-md" v-else>
          <div
            v-for="servicio in obtenerServiciosFiltrados()"
            :key="servicio.id"
            class="col-12 col-sm-6 col-md-4"
          >
            <q-card
              flat
              bordered
              :class="{
                'bg-red-1': servicio.estadoPago === 'pendiente',
                'bg-orange-1': servicio.estadoPago === 'abono',
                'bg-white': servicio.estadoPago === 'pagado'
              }"
              class="shadow-1"
            >
              <!-- Cabecera de la Tarjeta -->
              <q-card-section class="q-pb-xs">
                <div class="row items-center no-wrap">
                  <div class="col">
                    <div class="text-subtitle1 text-weight-bold text-uppercase">
                      {{ servicio.cliente }}
                    </div>
                    <div class="text-caption text-grey-8">
                      <q-icon name="smartphone" /> {{ servicio.equipo }}
                    </div>
                  </div>
                  <div class="col-auto">
                    <q-chip
                      dense
                      text-color="white"
                      :color="obtenerColorEstadoEquipo(servicio.estadoEquipo)"
                      :icon="obtenerIconoEstadoEquipo(servicio.estadoEquipo)"
                    >
                      {{ servicio.estadoEquipo }}
                    </q-chip>
                  </div>
                </div>
              </q-card-section>

              <q-separator />

              <!-- Detalle de la Reparación -->
              <q-card-section class="q-py-sm">
                <div class="text-body2 text-weight-medium text-primary q-mb-xs">
                  <q-icon name="build" /> {{ servicio.tipoReparacion }}
                </div>
                
                <div class="row text-caption text-grey-7 q-col-gutter-xs">
                  <div class="col-6">
                    <q-icon name="person" /> <strong>Técnico:</strong> {{ servicio.tecnico }}
                  </div>
                  <div class="col-6">
                    <q-icon name="event" /> {{ servicio.fechaRecepcion }}
                  </div>
                </div>

                <div v-if="servicio.observaciones" class="q-mt-sm text-caption bg-grey-3 q-pa-xs rounded-borders text-italic">
                  "{{ servicio.observaciones }}"
                </div>
              </q-card-section>

              <q-separator />

              <!-- Estado de Pago -->
              <q-card-section class="q-py-xs">
                <div class="row items-center justify-between">
                  <div>
                    <span class="text-caption text-grey-7">Precio: </span>
                    <span class="text-subtitle2 text-weight-bolder text-green-9">
                      ${{ Number(servicio.precio).toLocaleString('es-CO') }}
                    </span>
                  </div>
                  <div>
                    <q-badge :color="obtenerColorEstadoPago(servicio.estadoPago)">
                      {{ servicio.estadoPago.toUpperCase() }} ({{ servicio.metodoPago }})
                    </q-badge>
                  </div>
                </div>
              </q-card-section>

              <!-- Rating (Solo entregados) -->
              <q-card-section v-if="servicio.estadoEquipo === 'Entregado'" class="q-py-xs bg-grey-2">
                <div class="row items-center justify-between">
                  <span class="text-caption text-grey-8">Calificación:</span>
                  <q-rating
                    v-model="servicio.calificacion"
                    size="18px"
                    color="amber"
                    icon="star_border"
                    icon-selected="star"
                    readonly
                  />
                </div>
              </q-card-section>

              <!-- Botones de Acción -->
              <q-card-actions align="right" class="q-pt-none">
                <q-btn
                  flat
                  round
                  color="primary"
                  icon="edit"
                  size="sm"
                  @click="abrirModalEditar(servicio)"
                >
                  <q-tooltip>Editar</q-tooltip>
                </q-btn>
                <q-btn
                  flat
                  round
                  color="negative"
                  icon="delete"
                  size="sm"
                  @click="solicitarEliminacion(servicio)"
                >
                  <q-tooltip>Eliminar</q-tooltip>
                </q-btn>
              </q-card-actions>
            </q-card>
          </div>
        </div>

        <!-- Modal de Formulario (Crear / Editar) -->
        <q-dialog v-model="mostrarModal" persistent>
          <q-card style="width: 100%; max-width: 500px;">
            <q-card-section class="bg-primary text-white row items-center justify-between">
              <div class="text-h6">
                {{ modoEdicion ? 'Editar Servicio Técnico' : 'Nuevo Servicio Técnico' }}
              </div>
              <q-btn icon="close" flat round dense v-close-popup />
            </q-card-section>

            <q-form @submit="guardarServicio" class="q-gutter-xs">
              <q-card-section class="q-pa-md scroll" style="max-height: 70vh;">
                <!-- Cliente -->
                <q-input
                  v-model="formulario.cliente"
                  label="Nombre del Cliente *"
                  outlined
                  dense
                  class="q-mb-sm"
                  :rules="[val => !!val || 'El nombre es obligatorio']"
                />

                <!-- Marca y Modelo -->
                <q-input
                  v-model="formulario.equipo"
                  label="Marca y Modelo del Equipo *"
                  placeholder="Ej: iPhone 12, Samsung A15"
                  outlined
                  dense
                  class="q-mb-sm"
                  :rules="[val => !!val || 'El equipo es obligatorio']"
                />

                <!-- Tipo de Reparación -->
                <q-select
                  v-model="formulario.tipoReparacion"
                  :options="opcionesReparacion"
                  label="Tipo de Reparación *"
                  outlined
                  dense
                  class="q-mb-sm"
                  :rules="[val => !!val || 'Seleccione un tipo de reparación']"
                />

                <!-- Técnico -->
                <q-select
                  v-model="formulario.tecnico"
                  :options="opcionesTecnicos"
                  label="Técnico que Atendió *"
                  outlined
                  dense
                  class="q-mb-sm"
                  :rules="[val => !!val || 'Seleccione un técnico']"
                />

                <!-- Fecha y Hora -->
                <q-input
                  v-model="formulario.fechaRecepcion"
                  label="Fecha y Hora de Recepción *"
                  type="datetime-local"
                  outlined
                  dense
                  stack-label
                  class="q-mb-sm"
                  :rules="[val => !!val || 'La fecha es obligatoria']"
                />

                <!-- Precio -->
                <q-input
                  v-model.number="formulario.precio"
                  label="Precio Cobrado ($) *"
                  type="number"
                  outlined
                  dense
                  class="q-mb-sm"
                  :rules="[
                    val => val !== null && val !== '' || 'El precio es obligatorio',
                    val => val >= 0 || 'El precio no puede ser negativo'
                  ]"
                />

                <!-- Método de Pago -->
                <q-select
                  v-model="formulario.metodoPago"
                  :options="opcionesMetodoPago"
                  label="Método de Pago *"
                  outlined
                  dense
                  class="q-mb-sm"
                  :rules="[val => !!val || 'Seleccione un método de pago']"
                />

                <!-- Estado del Pago -->
                <q-select
                  v-model="formulario.estadoPago"
                  :options="opcionesEstadoPago"
                  label="Estado del Pago *"
                  outlined
                  dense
                  emit-value
                  map-options
                  class="q-mb-sm"
                  :rules="[val => !!val || 'Seleccione el estado del pago']"
                />

                <!-- Estado del Equipo -->
                <q-select
                  v-model="formulario.estadoEquipo"
                  :options="opcionesEstadoEquipo"
                  label="Estado del Equipo *"
                  outlined
                  dense
                  class="q-mb-sm"
                  :rules="[val => !!val || 'Seleccione el estado del equipo']"
                />

                <!-- Calificación -->
                <div v-if="formulario.estadoEquipo === 'Entregado'" class="q-mb-sm bg-blue-1 q-pa-sm rounded-borders">
                  <div class="text-caption text-weight-bold text-grey-8 q-mb-xs">Calificación del cliente (1 a 5 estrellas):</div>
                  <q-rating
                    v-model="formulario.calificacion"
                    size="28px"
                    color="amber"
                    icon="star_border"
                    icon-selected="star"
                  />
                </div>

                <!-- Observaciones -->
                <q-input
                  v-model="formulario.observaciones"
                  label="Observaciones (Opcional)"
                  type="textarea"
                  outlined
                  dense
                  rows="2"
                  placeholder="Ej: Pantalla partida en la esquina, no prende, etc."
                />
              </q-card-section>

              <q-card-actions align="right" class="bg-grey-1 q-pa-md">
                <q-btn flat label="Cancelar" color="grey-8" v-close-popup />
                <q-btn unelevated label="Guardar" color="primary" type="submit" />
              </q-card-actions>
            </q-form>
          </q-card>
        </q-dialog>

        <!-- Modal de Confirmación para Eliminar -->
        <q-dialog v-model="mostrarModalEliminar" persistent>
          <q-card>
            <q-card-section class="row items-center">
              <q-avatar icon="warning" color="negative" text-color="white" />
              <span class="q-ml-sm text-body1">
                ¿Está seguro de eliminar el servicio de <strong>{{ servicioAEliminar ? servicioAEliminar.cliente : '' }}</strong> ({{ servicioAEliminar ? servicioAEliminar.equipo : '' }})?
              </span>
            </q-card-section>

            <q-card-actions align="right">
              <q-btn flat label="Cancelar" color="grey" v-close-popup />
              <q-btn unelevated label="Sí, Eliminar" color="negative" @click="confirmarEliminar" />
            </q-card-actions>
          </q-card>
        </q-dialog>

         <q-footer elevated>
      <q-toolbar>
        <q-toolbar-title 
        style="text-align: center; font-family: Tangerine; font-size: 14px; color: #ffffff;">
          Taller de Don Efraín-Especialista en Reparación de Electrónica</q-toolbar-title>
      </q-toolbar>
    </q-footer>

      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

// Persistencia de datos local
const servicios = useLocalStorage('taller_servicios', [
  {
    id: 1,
    cliente: 'Carlos Mendoza',
    equipo: 'Samsung A15',
    tipoReparacion: 'Cambio de pantalla',
    tecnico: 'Efraín Campos',
    fechaRecepcion: '2026-08-20T10:30',
    precio: 120000,
    metodoPago: 'Efectivo',
    estadoPago: 'pendiente',
    estadoEquipo: 'Recibido',
    calificacion: 0,
    observaciones: 'Pantalla con líneas verdes, táctil no responde.'
  },
  {
    id: 2,
    cliente: 'María Rojas',
    equipo: 'iPhone 12',
    tipoReparacion: 'Cambio de batería',
    tecnico: 'Juan Pérez',
    fechaRecepcion: '2026-08-21T14:15',
    precio: 180000,
    metodoPago: 'Transferencia',
    estadoPago: 'pagado',
    estadoEquipo: 'Entregado',
    calificacion: 5,
    observaciones: 'Batería al 72% de condición.'
  },
  {
    id: 3,
    cliente: 'Luis Fernández',
    equipo: 'Motorola G50',
    tipoReparacion: 'Mantenimiento de software',
    tecnico: 'Camilo Gómez',
    fechaRecepcion: '2026-08-22T09:45',
    precio: 80000,
    metodoPago: 'Tarjeta',
    estadoPago: 'abono',
    estadoEquipo: 'En reparación',
    calificacion: 0,
    observaciones: 'El equipo se reinicia solo, necesita actualización.'
  }
])

// Estados reactivos
const mostrarModal = ref(false)
const modoEdicion = ref(false)
const idEdicion = ref(null)

const mostrarModalEliminar = ref(false)
const servicioAEliminar = ref(null)

const filtroTexto = ref('')
const filtroEstado = ref('todos')

// Objeto del formulario
const formulario = ref({
  cliente: '',
  equipo: '',
  tipoReparacion: '',
  tecnico: '',
  fechaRecepcion: '',
  precio: null,
  metodoPago: '',
  estadoPago: 'pendiente',
  estadoEquipo: 'Recibido',
  calificacion: 0,
  observaciones: ''
})

// Opciones de selects
const opcionesReparacion = [
  'Cambio de pantalla',
  'Cambio de batería',
  'Cambio de pin de carga',
  'Liberación',
  'Mantenimiento de software',
  'Cambio de flex',
  'Cambio de cámara',
  'Reparación de lente',
  'Otros'
]

const opcionesTecnicos = ['Efraín Campos', 'Juan Pérez', 'Camilo Gómez']
const opcionesMetodoPago = ['Efectivo', 'Transferencia', 'Tarjeta']

const opcionesEstadoPago = [
  { label: 'Pagado', value: 'pagado' },
  { label: 'Pendiente', value: 'pendiente' },
  { label: 'Abono', value: 'abono' }
]

const opcionesEstadoEquipo = [
  'Recibido',
  'En reparación',
  'Listo para entregar',
  'Entregado'
]

const opcionesFiltroEstado = [
  { label: 'Todos los estados', value: 'todos' },
  { label: 'Recibido', value: 'Recibido' },
  { label: 'En reparación', value: 'En reparación' },
  { label: 'Listo para entregar', value: 'Listo para entregar' },
  { label: 'Entregado', value: 'Entregado' }
]

// Funciones de utilidad y control de modales
function obtenerFechaActualFormateada() {
  const ahora = new Date()
  const año = ahora.getFullYear()
  const mes = String(ahora.getMonth() + 1).padStart(2, '0')
  const dia = String(ahora.getDate()).padStart(2, '0')
  const horas = String(ahora.getHours()).padStart(2, '0')
  const minutos = String(ahora.getMinutes()).padStart(2, '0')
  return `${año}-${mes}-${dia}T${horas}:${minutos}`
}

function limpiarFormulario() {
  formulario.value = {
    cliente: '',
    equipo: '',
    tipoReparacion: '',
    tecnico: '',
    fechaRecepcion: obtenerFechaActualFormateada(),
    precio: null,
    metodoPago: 'Efectivo',
    estadoPago: 'pendiente',
    estadoEquipo: 'Recibido',
    calificacion: 0,
    observaciones: ''
  }
}

function abrirModalNuevo() {
  modoEdicion.value = false
  idEdicion.value = null
  limpiarFormulario()
  mostrarModal.value = true
}

function abrirModalEditar(servicio) {
  modoEdicion.value = true
  idEdicion.value = servicio.id
  formulario.value = { ...servicio }
  mostrarModal.value = true
}

function guardarServicio() {
  if (modoEdicion.value) {
    const index = servicios.value.findIndex(s => s.id === idEdicion.value)
    if (index !== -1) {
      servicios.value[index] = {
        ...formulario.value,
        id: idEdicion.value
      }
    }
  } else {
    const nuevoServicio = {
      ...formulario.value,
      id: Date.now()
    }
    servicios.value.push(nuevoServicio)
  }
  mostrarModal.value = false
}

function solicitarEliminacion(servicio) {
  servicioAEliminar.value = servicio
  mostrarModalEliminar.value = true
}

function confirmarEliminar() {
  if (servicioAEliminar.value) {
    servicios.value = servicios.value.filter(s => s.id !== servicioAEliminar.value.id)
    servicioAEliminar.value = null
  }
  mostrarModalEliminar.value = false
}

// Lógica de filtrado dinámico en la plantilla
function obtenerServiciosFiltrados() {
  return servicios.value.filter(servicio => {
    const texto = filtroTexto.value.toLowerCase().trim()
    const coincideTexto = !texto ||
      servicio.cliente.toLowerCase().includes(texto) ||
      servicio.equipo.toLowerCase().includes(texto) ||
      servicio.tecnico.toLowerCase().includes(texto)

    const coincideEstado = filtroEstado.value === 'todos' || servicio.estadoEquipo === filtroEstado.value

    return coincideTexto && coincideEstado
  })
}

// Métodos auxiliares para dashboard y renderizado
function contarPorEstado(tipo) {
  if (tipo === 'en_taller') {
    return servicios.value.filter(s => s.estadoEquipo === 'Recibido' || s.estadoEquipo === 'En reparación').length
  }
  if (tipo === 'listo') {
    return servicios.value.filter(s => s.estadoEquipo === 'Listo para entregar').length
  }
  return 0
}

function contarPagosPendientes() {
  return servicios.value.filter(s => s.estadoPago === 'pendiente' || s.estadoPago === 'abono').length
}

function obtenerColorEstadoEquipo(estado) {
  switch (estado) {
    case 'Recibido': return 'blue-7'
    case 'En reparación': return 'warning'
    case 'Listo para entregar': return 'positive'
    case 'Entregado': return 'grey-7'
    default: return 'primary'
  }
}

function obtenerIconoEstadoEquipo(estado) {
  switch (estado) {
    case 'Recibido': return 'inbox'
    case 'En reparación': return 'build'
    case 'Listo para entregar': return 'mark_email_read'
    case 'Entregado': return 'task_alt'
    default: return 'help'
  }
}

function obtenerColorEstadoPago(estado) {
  switch (estado) {
    case 'pagado': return 'positive'
    case 'pendiente': return 'negative'
    case 'abono': return 'warning'
    default: return 'grey'
  }
}
</script>