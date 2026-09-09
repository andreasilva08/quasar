<template>
  <q-layout view="lHh Lpr lFf" class="app-bg">
    
    <!-- HEADER-->
    <q-header class="custom-header">
      <q-toolbar class="q-py-md max-width-container">
        <div class="row items-center q-gutter-x-sm">
          <q-avatar size="44px" color="blue-10" text-color="white" class="shadow-3 icon-avatar">
            <q-icon name="build_circle" size="28px" />
          </q-avatar>
          <div>
            <div class="text-h6 text-weight-bolder brand-title">Taller Don Efraín</div>
            <div class="text-caption text-blue-2 font-sub">Gestión Integral de Servicio Técnico</div>
          </div>
        </div>

        <q-space />

        <!-- Botón Agregar  -->
        <q-btn
          icon="add_circle"
          label="Nuevo Servicio"
          unelevated
          size="md"
          class="btn-gradient text-weight-bold q-px-lg shadow-4"
          @click="abrirModalNuevo"
        />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="q-pa-md max-width-container">
        
        <!-- Tarjetas de Resumen Rápido -->
        <div class="row q-col-gutter-md q-mb-lg">
          <div class="col-6 col-sm-3">
            <q-card flat class="stat-card border-blue">
              <q-card-section class="q-pa-sm text-center">
                <div class="stat-label">Total Registrados</div>
                <div class="stat-value text-primary">{{ servicios.length }}</div>
              </q-card-section>
            </q-card>
          </div>
          <div class="col-6 col-sm-3">
            <q-card flat class="stat-card border-amber">
              <q-card-section class="q-pa-sm text-center">
                <div class="stat-label">Por Entregar</div>
                <div class="stat-value text-amber-9">
                  {{ servicios.filter(s => s.estadoEquipo !== 'Entregado').length }}
                </div>
              </q-card-section>
            </q-card>
          </div>
          <div class="col-6 col-sm-3">
            <q-card flat class="stat-card border-red">
              <q-card-section class="q-pa-sm text-center">
                <div class="stat-label">Pagos Pendientes</div>
                <div class="stat-value text-red-8">
                  {{ servicios.filter(s => s.estadoPago === 'Pendiente').length }}
                </div>
              </q-card-section>
            </q-card>
          </div>
          <div class="col-6 col-sm-3">
            <q-card flat class="stat-card border-green">
              <q-card-section class="q-pa-sm text-center">
                <div class="stat-label">Completados</div>
                <div class="stat-value text-green-8">
                  {{ servicios.filter(s => s.estadoEquipo === 'Entregado').length }}
                </div>
              </q-card-section>
            </q-card>
          </div>
        </div>

        <!-- BARRA DE BÚSQUEDA Y FILTROS -->
        <q-card flat class="filter-card q-pa-md q-mb-lg">
          <div class="row q-col-gutter-md items-center">
            <div class="col-12 col-md-7">
              <q-input
                v-model="textoBusqueda"
                placeholder="Buscar por cliente, marca, modelo o técnico..."
                outlined
                dense
                clearable
                class="custom-input"
              >
                <template v-slot:prepend>
                  <q-icon name="search" color="primary" />
                </template>
              </q-input>
            </div>
            <div class="col-12 col-sm-6 col-md-3">
              <q-select
                v-model="filtroEstado"
                :options="['Todos', 'Recibido', 'En reparación', 'Listo para entregar', 'Entregado']"
                label="Filtrar por estado"
                outlined
                dense
                class="custom-input"
              />
            </div>
            <div class="col-12 col-sm-6 col-md-2 text-right">
              <q-btn
                flat
                color="grey-8"
                icon="restart_alt"
                label="Limpiar"
                class="full-width btn-reset"
                @click="limpiarFiltros"
              />
            </div>
          </div>
        </q-card>

        <!-- Mensaje si no hay datos o la búsqueda no arroja resultados -->
        <div v-if="serviciosFiltrados().length === 0" class="empty-state text-center q-pa-xl">
          <q-icon name="search_off" size="64px" color="grey-5" />
          <div class="text-h6 q-mt-md text-grey-8 text-weight-bold">No se encontraron servicios</div>
          <p class="text-grey-6">
            {{ servicios.length === 0 ? 'Haga clic en "Nuevo Servicio" para registrar un equipo.' : 'Intente cambiando el término de búsqueda o filtro.' }}
          </p>
        </div>

        <!-- LISTADO DE TARJETAS DE SERVICIOS -->
        <div class="row q-col-gutter-md">
          <div
            v-for="servicio in serviciosFiltrados()"
            :key="servicio.id"
            class="col-12 col-sm-6 col-md-4"
          >
            <q-card
              flat
              class="service-card"
              :class="{
                'card-pending': servicio.estadoPago === 'Pendiente',
                'card-abono': servicio.estadoPago === 'Abono',
                'card-paid': servicio.estadoPago === 'Pagado'
              }"
            >
              <q-card-section class="q-pb-xs">
                <div class="row items-center no-wrap">
                  <div class="col">
                    <div class="text-subtitle1 text-weight-bolder text-primary ellipsis">
                      {{ servicio.cliente }}
                    </div>
                    <div class="text-subtitle2 text-weight-bold text-grey-8">
                      {{ servicio.marca }} - {{ servicio.modelo }}
                    </div>
                  </div>
                  <q-avatar
                    :color="obtenerColorEstado(servicio.estadoEquipo)"
                    text-color="white"
                    size="40px"
                    class="shadow-2"
                  >
                    <q-icon :name="obtenerIconoEstado(servicio.estadoEquipo)" size="22px" />
                  </q-avatar>
                </div>
              </q-card-section>

              <q-separator class="q-my-xs opacity-50" />

              <q-card-section class="q-py-sm text-body2">
                <div class="q-mb-xs"><strong>Reparación:</strong> {{ servicio.reparacion }}</div>
                <div class="q-mb-xs"><strong>Técnico:</strong> {{ servicio.tecnico }}</div>
                <div class="q-mb-xs text-caption text-grey-7">
                  <q-icon name="event" size="14px" class="q-mr-xs" />
                  {{ servicio.fecha }}
                </div>

                <div class="q-mt-sm q-mb-xs">
                  <strong>Precio:</strong> ${{ servicio.precio ? servicio.precio.toLocaleString() : 0 }}
                  <span v-if="servicio.estadoPago === 'Abono'" class="text-orange-10 text-weight-bold">
                    (Abonó: ${{ servicio.valorAbono ? servicio.valorAbono.toLocaleString() : 0 }})
                  </span>
                </div>

                <div class="q-mb-xs"><strong>Método de pago:</strong> {{ servicio.metodoPago }}</div>

                <div class="q-mt-md row items-center q-gutter-x-xs">
                  <q-badge :color="obtenerColorEstado(servicio.estadoEquipo)" class="badge-custom">
                    {{ servicio.estadoEquipo }}
                  </q-badge>
                  <q-badge :color="servicio.estadoPago === 'Pagado' ? 'green-7' : (servicio.estadoPago === 'Abono' ? 'orange-9' : 'red-7')" class="badge-custom">
                    {{ servicio.estadoPago }}
                  </q-badge>
                </div>

                <!-- Calificación si ya está entregado -->
                <div v-if="servicio.estadoEquipo === 'Entregado' && servicio.calificacion" class="q-mt-sm row items-center bg-grey-2 q-pa-xs rounded-borders">
                  <span class="text-caption text-grey-8 q-mr-xs">Calificación:</span>
                  <q-rating v-model="servicio.calificacion" readonly size="1.2em" color="amber" />
                </div>

                <!-- Observaciones -->
                <div v-if="servicio.observaciones" class="q-mt-xs text-caption text-grey-8 italic bg-amber-1 q-pa-xs rounded-borders">
                  <strong>Obs:</strong> {{ servicio.observaciones }}
                </div>
              </q-card-section>

              <q-separator class="opacity-50" />

              <!-- Acciones desactivadas en estado Entregado -->
              <q-card-actions align="right" class="q-px-md">
                <q-btn
                  flat
                  dense
                  color="primary"
                  icon="edit"
                  label="Editar"
                  :disable="servicio.estadoEquipo === 'Entregado'"
                  @click="abrirModalEditar(servicio)"
                />
                <q-btn
                  flat
                  dense
                  color="negative"
                  icon="delete"
                  label="Eliminar"
                  :disable="servicio.estadoEquipo === 'Entregado'"
                  @click="confirmarEliminar(servicio)"
                />
              </q-card-actions>
            </q-card>
          </div>
        </div>

        <!-- Botón flotante FAB para celulares -->
        <q-page-sticky position="bottom-right" :offset="[20, 20]">
          <q-btn fab icon="add" class="btn-gradient shadow-8" @click="abrirModalNuevo" />
        </q-page-sticky>

      </q-page>
    </q-page-container>

    <!-- FOOTER -->
    <q-footer class="custom-footer text-white q-py-md">
      <div class="max-width-container row items-center justify-between q-px-md">
        <div class="text-caption text-grey-4">
          © 2026 <strong>Taller Don Efraín</strong> — Sistema de Control de Servicio Técnico
        </div>
        <div class="text-caption text-grey-4 row items-center">
          <q-icon name="verified" size="16px" color="blue-4" class="q-mr-xs" />
          Versión 2.0 • Datos sincronizados localmente
        </div>
      </div>
    </q-footer>

    <!-- MODAL FORMULARIO DE REGISTRO / EDICIÓN -->
    <q-dialog v-model="modalFormulario" persistent>
      <q-card class="modal-card">
        <q-card-section class="row items-center modal-header text-white">
          <div class="text-h6 text-weight-bold">{{ esEdicion ? 'Editar Servicio' : 'Nuevo Servicio' }}</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>

        <q-card-section class="q-pa-lg">
          <q-form @submit="guardarServicio" class="form-grid">
            
            <q-input
  v-model="form.cliente"
  label="Nombre del cliente *"
  outlined
  dense
  hide-bottom-space
  :rules="[
    val => (val && val.trim().length > 0) || 'El nombre es obligatorio',
    val => /^[a-zA-ZáéíóúÁÉÍÓÚñÑ\s]+$/.test(val.trim()) || 'Solo se permiten letras y espacios'
  ]"
/>

            <div class="form-row">
              <div class="form-col">
                <q-select
                  v-model="form.marca"
                  :options="opcionesMarcas"
                  label="Marca *"
                  outlined
                  dense
                  hide-bottom-space
                  :rules="[val => !!val || 'Requerido']"
                />
              </div>
              <div class="form-col">
                <q-input
                  v-model="form.modelo"
                  label="Modelo *"
                  outlined
                  dense
                  hide-bottom-space
                  placeholder="Ej: Redmi Note 10"
                  :rules="[val => !!val || 'Requerido']"
                />
              </div>
            </div>

            <q-select
              v-model="form.reparacion"
              :options="opcionesReparacion"
              label="Tipo de reparación *"
              outlined
              dense
              hide-bottom-space
              :rules="[val => !!val || 'Requerido']"
            />

            <q-select
              v-model="form.tecnico"
              :options="opcionesTecnicos"
              label="Técnico que atendió *"
              outlined
              dense
              hide-bottom-space
              :rules="[val => !!val || 'Requerido']"
            />

            <q-input
              v-model="form.fecha"
              label="Fecha y hora de recepción"
              outlined
              dense
              disable
              readonly
            />

            <div class="form-row">
              <div class="form-col">
                <q-input
                  v-model.number="form.precio"
                  type="number"
                  label="Precio cobrado ($) *"
                  outlined
                  dense
                  hide-bottom-space
                  :rules="[val => (val !== null && val !== '' && val > 0) || 'Monto inválido']"
                />
              </div>
              <div class="form-col">
                <q-select
                  v-model="form.metodoPago"
                  :options="opcionesMetodoPago"
                  label="Método de pago *"
                  outlined
                  dense
                  hide-bottom-space
                  :rules="[val => !!val || 'Requerido']"
                />
              </div>
            </div>

            <div class="form-row">
              <div class="form-col">
                <q-select
                  v-model="form.estadoPago"
                  :options="opcionesEstadoPago"
                  label="Estado del pago *"
                  outlined
                  dense
                  hide-bottom-space
                  :rules="[val => !!val || 'Requerido']"
                />
              </div>
              <div class="form-col" v-if="form.estadoPago === 'Abono'">
                <q-input
                  v-model.number="form.valorAbono"
                  type="number"
                  label="Valor del abono ($) *"
                  outlined
                  dense
                  hide-bottom-space
                  :rules="[
                    val => (val !== null && val !== '' && val > 0) || 'Abono inválido',
                    val => val <= form.precio || 'Excede el total'
                  ]"
                />
              </div>
            </div>

            <!-- Selector del Estado del Equipo con restricción de pago -->
<q-select
  v-model="form.estadoEquipo"
  :options="opcionesEstadoEquipo"
  label="Estado del equipo *"
  outlined
  dense
  hide-bottom-space
  :disable="!esEdicion"
  :option-disable="opt => opt === 'Entregado' && form.estadoPago !== 'Pagado'"
  :rules="[val => !!val || 'Requerido']"
>
  <template v-slot:option="scope">
    <q-item v-bind="scope.itemProps">
      <q-item-section>
        <q-item-label>{{ scope.opt }}</q-item-label>
        <q-item-label v-if="scope.opt === 'Entregado' && form.estadoPago !== 'Pagado'" caption class="text-red-8">
          (Requiere estado 'Pagado' para entregar)
        </q-item-label>
      </q-item-section>
    </q-item>
  </template>
</q-select>

            <q-input
              v-model="form.observaciones"
              label="Observaciones (Opcional)"
              type="textarea"
              outlined
              dense
              rows="2"
            />

            <div class="row justify-end q-mt-md q-gutter-x-sm">
              <q-btn label="Cancelar" color="grey-7" flat v-close-popup />
              <q-btn label="Guardar Registro" type="submit" class="btn-gradient" unelevated />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- MODAL CALIFICACIÓN AL ENTREGAR -->
    <q-dialog v-model="modalCalificacion" persistent>
      <q-card style="width: 380px; max-width: 90vw; border-radius: 16px;">
        <q-card-section class="bg-teal text-white">
          <div class="text-h6 text-weight-bold">Entrega de Equipo</div>
        </q-card-section>

        <q-card-section class="q-pa-md text-center">
          <p class="text-body1 text-grey-9 q-mb-sm">
            Ingrese la calificación del cliente (1 a 5 estrellas):
          </p>
          <q-rating v-model="calificacionCliente" size="2.5em" color="amber" icon="star" />
        </q-card-section>

        <q-card-actions align="right">
          <q-btn label="Cancelar" flat color="grey-7" @click="cancelarEntrega" />
          <q-btn
            label="Confirmar Entrega"
            color="teal"
            unelevated
            :disable="calificacionCliente === 0"
            @click="guardarEntregaConCalificacion"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- MODAL CONFIRMACIÓN DE ELIMINACIÓN -->
    <q-dialog v-model="modalEliminar" persistent>
      <q-card style="border-radius: 16px;">
        <q-card-section class="row items-center q-pa-md">
          <q-avatar icon="warning" color="negative" text-color="white" />
          <span class="q-ml-sm text-body1">¿Desea eliminar permanentemente este registro?</span>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Cancelar" color="grey-7" v-close-popup />
          <q-btn flat label="Eliminar" color="negative" @click="eliminarServicioConfirmado" />
        </q-card-actions>
      </q-card>
    </q-dialog>

  </q-layout>
</template>

<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

// Persistencia local
const servicios = useLocalStorage('taller_efrain_servicios', [])

// Filtros y Búsqueda
const textoBusqueda = ref('')
const filtroEstado = ref('Todos')

// Opciones predefinidas
const opcionesMarcas = ['Apple', 'Asus', 'Google', 'Honor', 'Huawei', 'iPhone', 'Motorola', 'Nokia', 'Oppo', 'Realme', 'Samsung', 'Xiaomi', 'Otra']
const opcionesReparacion = [
  'Cambio de batería', 
  'Cambio de pin de carga', 
  'Cambio de pantalla', 
  'Liberación', 
  'Mantenimiento de software', 
  'Reparación de cámara', 
  'Reparación de micrófono', 
  'Otros'
]
const opcionesTecnicos = ['Técnico Efraín Castrillón', 'Técnico Andrés Echeverry', 'Técnico Carlos Rodríguez']
const opcionesMetodoPago = ['Efectivo', 'Transferencia', 'Tarjeta']
const opcionesEstadoPago = ['Pagado', 'Pendiente', 'Abono']
const opcionesEstadoEquipo = ['Recibido', 'En reparación', 'Listo para entregar', 'Entregado']

// Modales
const modalFormulario = ref(false)
const modalEliminar = ref(false)
const modalCalificacion = ref(false)

// Variables reactivas de control
const esEdicion = ref(false)
const idServicioEliminar = ref(null)
const servicioAEntregar = ref(null)
const calificacionCliente = ref(0)

// Formulario
const form = ref({
  id: null,
  cliente: '',
  marca: null,
  modelo: '',
  reparacion: null,
  tecnico: null,
  fecha: '',
  precio: null,
  metodoPago: null,
  estadoPago: null,
  valorAbono: 0,
  estadoEquipo: 'Recibido',
  calificacion: null,
  observaciones: ''
})

// Lógica de búsqueda
const serviciosFiltrados = () => {
  return servicios.value.filter(servicio => {
    const texto = textoBusqueda.value.toLowerCase().trim()
    
    const coincideTexto = !texto ||
      (servicio.cliente && servicio.cliente.toLowerCase().includes(texto)) ||
      (servicio.marca && servicio.marca.toLowerCase().includes(texto)) ||
      (servicio.modelo && servicio.modelo.toLowerCase().includes(texto)) ||
      (servicio.tecnico && servicio.tecnico.toLowerCase().includes(texto))

    const coincideEstado = filtroEstado.value === 'Todos' || servicio.estadoEquipo === filtroEstado.value

    return coincideTexto && coincideEstado
  })
}

const limpiarFiltros = () => {
  textoBusqueda.value = ''
  filtroEstado.value = 'Todos'
}

// Fecha y hora automatizada en formato es-CO
const obtenerFechaHoraActual = () => {
  const ahora = new Date()
  return ahora.toLocaleString('es-CO', {
    dateStyle: 'short',
    timeStyle: 'short'
  })
}

const abrirModalNuevo = () => {
  esEdicion.value = false
  form.value = {
    id: Date.now(),
    cliente: '',
    marca: null,
    modelo: '',
    reparacion: null,
    tecnico: null,
    fecha: obtenerFechaHoraActual(),
    precio: null,
    metodoPago: null,
    estadoPago: null,
    valorAbono: 0,
    estadoEquipo: 'Recibido',
    calificacion: null,
    observaciones: ''
  }
  modalFormulario.value = true
}

const abrirModalEditar = (servicio) => {
  esEdicion.value = true
  form.value = { ...servicio }
  modalFormulario.value = true
}

const guardarServicio = () => {
  // RESTRICCIÓN CLAVE: No se puede entregar si el pago está en "Abono" o "Pendiente"
  if (form.value.estadoEquipo === 'Entregado' && form.value.estadoPago !== 'Pagado') {
    // Si intenta entregar sin haber pagado la totalidad, revertimos o bloqueamos
    alert('No se puede entregar un equipo con saldo pendiente. Debe cambiar el estado de pago a "Pagado" primero.')
    return
  }

  // Si pasa a estado Entregado y está Pagado, requerimos la calificación
  if (form.value.estadoEquipo === 'Entregado' && !form.value.calificacion) {
    servicioAEntregar.value = { ...form.value }
    calificacionCliente.value = 0
    modalFormulario.value = false
    modalCalificacion.value = true
    return
  }

  ejecutarGuardado(form.value)
}

const ejecutarGuardado = (datos) => {
  // Limpia espacios al inicio y al final del nombre del cliente
  if (datos.cliente) {
    datos.cliente = datos.cliente.trim()
  }

  if (datos.estadoPago !== 'Abono') {
    datos.valorAbono = 0
  }

  if (esEdicion.value) {
    const index = servicios.value.findIndex(s => s.id === datos.id)
    if (index !== -1) {
      servicios.value[index] = { ...datos }
    }
  } else {
    servicios.value.push({ ...datos })
  }
  modalFormulario.value = false
}

const guardarEntregaConCalificacion = () => {
  servicioAEntregar.value.calificacion = calificacionCliente.value
  ejecutarGuardado(servicioAEntregar.value)
  modalCalificacion.value = false
  servicioAEntregar.value = null
}

const cancelarEntrega = () => {
  modalCalificacion.value = false
  servicioAEntregar.value = null
}

const confirmarEliminar = (servicio) => {
  idServicioEliminar.value = servicio.id
  modalEliminar.value = true
}

const eliminarServicioConfirmado = () => {
  servicios.value = servicios.value.filter(s => s.id !== idServicioEliminar.value)
  modalEliminar.value = false
  idServicioEliminar.value = null
}

const obtenerIconoEstado = (estado) => {
  switch (estado) {
    case 'Recibido': return 'inbox'
    case 'En reparación': return 'build'
    case 'Listo para entregar': return 'mark_email_read'
    case 'Entregado': return 'task_alt'
    default: return 'help'
  }
}

const obtenerColorEstado = (estado) => {
  switch (estado) {
    case 'Recibido': return 'blue-7'
    case 'En reparación': return 'orange-8'
    case 'Listo para entregar': return 'teal'
    case 'Entregado': return 'grey-7'
    default: return 'primary'
  }
}
</script>
