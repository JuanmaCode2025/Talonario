<template>
  <div class="app-container" :style="customStyles">
    <div v-if="showWelcomeModal" class="welcome-modal-container">
      <div class="welcome-modal-content">
        <h2>¡Bienvenidos!</h2>
        <p>Gracias por visitar nuestra página de sorteos</p>
        <button @click="openDataModal">
          <span>Crear Fila</span>
          <span>➕</span>
        </button>
      </div>
    </div>

    <div v-if="showDataModal" class="form-modal-container">
      <div class="form-modal-content">
        <div class="modal-header">
          <h3>Ingresar Datos del Sorteo</h3>
        </div>

        <div v-if="!datosGuardados" class="formulario-sorteo">
          <div class="form-group">
            <label>Ingrese la Cantidad del Premio</label>
            <div class="input-with-icon">
              <span>🏆</span>
              <input
                v-model="premioDisplay"
                type="text"
                placeholder="No se valores Negativos"
                @input="formatCurrency($event, 'premio')"
              />
            </div>
          </div>

          <div class="form-group">
            <label>Ingrese el Valor de la Boleta</label>
            <div class="input-with-icon">
              <span>🎫</span>
              <input
                v-model="precioUnitarioDisplay"
                type="text"
                placeholder="No se valores Negativos"
                @input="formatCurrency($event, 'precioUnitario')"
              />
            </div>
          </div>

          <div class="form-group">
            <label>Tipo de Lotería</label>
            <div class="select-with-icon">
              <span>🎰</span>
              <select v-model="sorteo">
                <option disabled value="">Seleccione el tipo de loteria</option>
                <option>Baloto</option>
                <option>Chances</option>
                <option>Superastro</option>
                <option>Raspa y Gana</option>
              </select>
            </div>
          </div>

          <div class="form-group">
            <label>Cantidad de Boletas</label>
            <div class="select-with-icon">
              <span>📚</span>
              <select v-model.number="cantidadBoletas">
                <option disabled value="">Seleccione cantidad</option>
                <option value="100">0 - 99</option>
                <option value="1000">0 - 999</option>
              </select>
            </div>
          </div>

          <div class="form-group">
            <label>Fecha del Sorteo</label>
            <div class="input-with-icon">
              <span>📅</span>
              <input v-model="fechaSorteo" type="date" />
            </div>
          </div>

          <div class="form-actions">
            <button @click="guardarDatos" class="primary-btn">
              <span>💾</span>
              <span>Guardar Sorteo</span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <div v-if="showAssignTicketModal" class="modal-overlay">
      <div
        class="modal-container"
        :class="`${boletas[boletaSeleccionada]?.status || 'available'}-modal`"
      >
        <div class="modal-header">
          <h3>
            Asignar Boleta #{{
              boletaSeleccionada.toString().padStart(boletaDigits, "0")
            }}
          </h3>
          <span class="modal-status">{{
            boletas[boletaSeleccionada]?.status || "available"
          }}</span>
          <button @click="closeAssignTicketModal" class="close-btn">
            <span>✕</span>
          </button>
        </div>
        <div class="modal-body">
          <div
            v-if="
              boletas[boletaSeleccionada]?.status !== 'available' && !isEditing
            "
          >
            <div class="participant-info">
              <h4>Datos del Participante</h4>
              <div class="info-grid">
                <div class="info-item">
                  <span class="info-label">Nombre:</span>
                  <span class="info-value">{{
                    boletas[boletaSeleccionada]?.nombre
                  }}</span>
                </div>
                <div class="info-item">
                  <span class="info-label">Teléfono:</span>
                  <span class="info-value">{{
                    boletas[boletaSeleccionada]?.telefono
                  }}</span>
                </div>
                <div class="info-item">
                  <span class="info-label">Direccion:</span>
                  <span class="info-value">{{
                    boletas[boletaSeleccionada]?.direccion
                  }}</span>
                </div>
              </div>
            </div>
            <div class="payment-info">
              <p v-if="boletas[boletaSeleccionada]?.status === 'reserved'">
                <span>Pendiente por Pagar:</span>
                <span
                  >₡{{
                    formatDecimal(
                      precioUnitario -
                        (boletas[boletaSeleccionada]?.montoPagado || 0)
                    )
                  }}</span
                >
              </p>
            </div>
          </div>

          <div
            v-if="
              boletas[boletaSeleccionada]?.status === 'available' || isEditing
            "
            class="form-container"
          >
            <div class="form-group">
              <label for="nombre">Nombre del Participante</label>
              <div class="input-with-icon">
                <span>👤</span>
                <input
                  type="text"
                  id="nombre"
                  v-model="currentTicketData.nombre"
                  placeholder="Nombre completo"
                />
              </div>
            </div>
            <div class="form-group">
              <label for="telefono">Teléfono</label>
              <div class="input-with-icon">
                <span>📱</span>
                <input
                  type="text"
                  id="telefono"
                  v-model="currentTicketData.telefono"
                  placeholder="Número de teléfono"
                  maxlength="10"
                  inputmode="numeric"
                  @input="
                    currentTicketData.telefono = currentTicketData.telefono
                      .replace(/\D/g, '')
                      .slice(0, 10)
                  "
                />
              </div>
            </div>
            <div class="form-group">
              <label for="direccion">Direccion</label>
              <div class="input-with-icon">
                <span>🌏</span>
                <input
                  type="text"
                  id="direccion"
                  v-model="currentTicketData.direccion"
                  placeholder="Direccion"
                />
              </div>
            </div>
          </div>
          <div
            v-if="isEditing"
            style="display: flex; gap: 10px; justify-content: end"
          >
            <button @click="isEditing = false" class="btn secondary">
              <span>✕</span>
              <span>Cancelar</span>
            </button>
            <button @click="saveEditedData" class="btn success">
              <span>💾</span>
              <span>Guardar Cambios</span>
            </button>
          </div>
          <div class="action-buttons">
            ;

            <div style="display: flex; gap: 10px">
              <!--v-else -->
              <button
                v-if="boletas[boletaSeleccionada]?.status === 'available'"
                @click="assignTicket('reserved')"
                class="btn secondary"
              >
                <span>🔖</span>
                <span>Apartar</span>
              </button>
              <button
                v-if="boletas[boletaSeleccionada]?.status === 'reserved'"
                @click="markAsPaid"
                class="btn success"
              >
                <span>✅</span>
                <span>Marcar como Pagada</span>
              </button>
              <button
                v-if="boletas[boletaSeleccionada]?.status !== 'available'"
                @click="enableEditMode"
                class="btn primary"
              >
                <span>✏️</span>
                <span>Editar Datos</span>
              </button>
              <button
                v-if="boletas[boletaSeleccionada]?.status === 'available'"
                @click="assignTicket('paid')"
                class="btn primary"
              >
                <span>💲</span>
                <span>Asignar y Pagar</span>
              </button>
              <button
                v-if="boletas[boletaSeleccionada]?.status !== 'available'"
                @click="releaseTicket"
                class="btn danger"
              >
                <span>↩️</span>
                <span>Liberar Boleta</span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="showTicketsTableModal" class="modal-overlay">
      <div class="modal-container-tabla">
        <div class="modal-header">
          <h3>Tabla de Boletas Asignadas</h3>
          <button @click="closeTicketsTableModal" class="close-btn">
            <span>✕</span>
          </button>
        </div>
        <div class="modal-body">
          <div class="ticket-table-container">
            <table>
              <thead>
                <tr>
                  <th># Boleta</th>
                  <th>Estado</th>
                  <th>Nombre</th>
                  <th>Teléfono</th>
                  <th>Direccion</th>
                  <th>Monto Pagado</th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="(ticket, index) in sortedBoletasForTable"
                  :key="index"
                  :class="ticket.status"
                >
                  <td>#{{ ticket.id }}</td>
                  <td>{{ ticket.status }}</td>
                  <td>{{ ticket.nombre }}</td>
                  <td>{{ ticket.telefono }}</td>
                  <td>{{ ticket.direccion }}</td>
                  <td>${{ formatDecimal(ticket.montoPagado) }}</td>
                </tr>
              </tbody>
              <tbody>
                <tr>
                  <td></td>
                  <td></td>
                  <td>Total :</td>
                  <td>{{ SumaMontosPagados }}</td>
                  <td></td>
                  <td></td>
                </tr>
                <tr>
                  <td></td>
                  <td></td>
                  <td>Premio :</td>
                  <td>{{ premio }}</td>
                  <td></td>
                  <td></td>
                </tr>
              </tbody>
            </table>
          </div>
          <div class="table-actions">
            <button @click="downloadTicketsPDF" class="btn primary">
              <span>⬇️</span>
              <span>Descargar PDF</span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <div v-if="showSettingsModal" class="modal-overlay">
      <div class="modal-container">
        <div class="modal-header">
          <h3>Configuración de la Aplicación</h3>
          <button @click="closeSettingsModal" class="close-btn">
            <span>✕</span>
          </button>
        </div>
        <div class="modal-body">
          <div class="form-group">
            <label for="primaryColor">Color Principal</label>
            <input
              type="color"
              id="primaryColor"
              v-model="themeColors.primary"
            />
          </div>
          <div class="form-group">
            <label for="secondaryColor">Color Secundario</label>
            <input
              type="color"
              id="secondaryColor"
              v-model="themeColors.secondary"
            />
          </div>
          <div class="form-group">
            <label for="successColor">Color Éxito</label>
            <input
              type="color"
              id="successColor"
              v-model="themeColors.success"
            />
          </div>
          <div class="form-group">
            <label for="dangerColor">Color Peligro</label>
            <input type="color" id="dangerColor" v-model="themeColors.danger" />
          </div>
          <div class="form-group">
            <label for="headerBgColor">Color de Fondo del Encabezado</label>
            <input
              type="color"
              id="headerBgColor"
              v-model="themeColors.headerBg"
            />
          </div>
          <div class="form-group">
            <label for="headerTextColor">Color de Texto del Encabezado</label>
            <input
              type="color"
              id="headerTextColor"
              v-model="themeColors.headerText"
            />
          </div>
          <div class="action-buttons">
            <button @click="saveThemeSettings" class="btn primary">
              <span>💾</span>
              <span>Guardar Configuración</span>
            </button>
            <button @click="resetThemeSettings" class="btn secondary">
              <span>↩️</span>
              <span>Restablecer</span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <div class="main-panel" v-if="!showWelcomeModal && !showDataModal">
      <header
        class="panel-header"
        :style="{
          backgroundColor: themeColors.headerBg,
          color: themeColors.headerText,
        }"
      >
        <div class="header-left" style="padding-top: 10px">
          <h1>{{ sorteo || "Nombre de la Rifa" }}</h1>
          <div
            class="badge active"
            :style="{ backgroundColor: themeColors.success }"
          >
            Activo
          </div>
        </div>

        <!-- Botón hamburguesa visible solo en móvil -->
        <button class="hamburger-btn" @click="showMenu = !showMenu">☰</button>

        <!-- Menú visible solo en móvil -->
        <div class="mobile-menu" v-if="showMenu">
          <div id="title ">
            <span>Talonario</span>
          </div>
          <button style="border-radu" @click="openSettingsModal">
            <span>⚙️</span>
          </button>
        </div>

        <!-- Menú para pantallas grandes -->
        <div class="header-center">
          <div id="title">
            <span>Talonario</span>
          </div>
        </div>
        <div class="header-right">
          <button @click="openSettingsModal">⚙️</button>
        </div>
      </header>

      <div class="body-principal">
        <div class="information">
          <div v-if="datosGuardados" class="data-card">
            <div class="card-header">
              <h4>Informacion del Sorteo</h4>
              <div
                class="badge success"
                :style="{ backgroundColor: themeColors.success }"
              >
                Guardado
              </div>
            </div>
            <div class="card-body">
              <div class="data-row">
                <span class="data-label">Premio:</span>
                <span class="data-value">${{ formatDecimal(premio) }}</span>
              </div>
              <div class="data-row">
                <span class="data-label">Valor boleta:</span>
                <span class="data-value"
                  >${{ formatDecimal(precioUnitario) }}</span
                >
              </div>
              <div class="data-row">
                <span class="data-label">Lotería:</span>
                <span class="data-value">{{ sorteo }}</span>
              </div>
              <div class="data-row">
                <span class="data-label">Cantidad de boletas:</span>
                <span class="data-value">{{ cantidadBoletas }}</span>
              </div>
              <div class="data-row">
                <span class="data-label">Fecha del sorteo:</span>
                <span class="data-value">{{ formatDate(fechaSorteo) }}</span>
              </div>
            </div>
            <div class="card-footer">
              <button class="secondary-btn" @click="editData">
                <span>✏️</span>
                <span>Editar Datos</span>
              </button>
            </div>
          </div>
          <div class="analis">
            <div class="card-header">
              <h4>Estadísticas Del talonario</h4>
            </div>

            <section class="grid_card">
              <div class="card">
                <h2>{{ cantidadBoletas }}</h2>
                <p>Total de Boletas</p>
              </div>
              <div class="card" id="disponible">
                <h2>{{ availableTicketsCount }}</h2>
                <p>Disponibles</p>
              </div>
              <div class="card" id="apartadas">
                <h2>{{ reservedTicketsCount }}</h2>
                <p>Apartadas</p>
              </div>
              <div class="card" id="pagadas">
                <h2>{{ paidTicketsCount }}</h2>
                <p>Pagadas</p>
              </div>
            </section>
            <div class="table-button-container">
              <button @click="openTicketsTableModal" class="btn info">
                <span>📊</span>
                <span>Ver Tabla de Boletas</span>
              </button>
            </div>
          </div>
          
        </div>

        <div class="talonarionumber">
          <button
            v-for="i in Array.from(
              { length: cantidadBoletas },
              (_, index) => index
            )"
            :key="i"
            :class="['number', boletas[i]?.status || 'available']"
            @click="openAssignTicketModal(i)"
          >
            #{{ i.toString().padStart(boletaDigits, "0") }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import Swal from "sweetalert2";
import jsPDF from "jspdf";
import "jspdf-autotable";

// const numberWin = ref(0)
// const showAnimation = ref(false)
// const isGenerating = ref(false)

// Suponiendo que cantidadBoletas viene de props o store
 // Cambia esto según tu implementación

// function generarnumber() {
//   if (cantidadBoletas.value <= 0) return
  
//   isGenerating.value = true
//   showAnimation.value = true
  
//   // Efecto de cuenta regresiva
//   let counter = 0
//   const interval = setInterval(() => {
//     numberWin.value = Math.floor(Math.random() * cantidadBoletas.value) + 1
//     counter++
    
//     if (counter > 10) {
//       clearInterval(interval)
//       isGenerating.value = false
//       setTimeout(() => showAnimation.value = false, 2000)
//     }
//   }, 100)
// }

// --- Estado de los modales ---
const showWelcomeModal = ref(true);
const showDataModal = ref(false);
const showAssignTicketModal = ref(false);
const showTicketsTableModal = ref(false);
const showSettingsModal = ref(false);
const isEditing = ref(false);

// --- Datos del sorteo ---
const premio = ref(0);
const premioDisplay = ref("");
const precioUnitario = ref(0);
const precioUnitarioDisplay = ref("");
const sorteo = ref("");
const cantidadBoletas = ref(0);
const fechaSorteo = ref("");
const datosGuardados = ref(false);

// --- Variables para la asignación de boletas ---
const boletaSeleccionada = ref(null);
const boletas = ref({});
const boletaDigits = ref(2);

const currentTicketData = ref({
  nombre: "",
  telefono: "",
  direccion: "",
  montoPagado: 0,
  montoPagadoDisplay: "",
});

// --- Theme Customization ---
const defaultThemeColors = {
  primary: "#3498db",
  secondary: "#95a5a6",
  success: "#2ecc71",
  danger: "#e74c3c",
  headerBg: "white",
  headerText: "#333",
};
const themeColors = ref({ ...defaultThemeColors });

const customStyles = computed(() => {
  return {
    "--primary-color": themeColors.value.primary,
    "--secondary-color": themeColors.value.secondary,
    "--success-color": themeColors.value.success,
    "--danger-color": themeColors.value.danger,
    "--header-bg-color": themeColors.value.headerBg,
    "--header-text-color": themeColors.value.headerText,
  };
});

// --- Computed properties for statistics ---
const availableTicketsCount = computed(() => {
  return (
    Object.values(boletas.value).filter((b) => b.status === "available")
      .length +
    (cantidadBoletas.value - Object.keys(boletas.value).length)
  );
});

const reservedTicketsCount = computed(() => {
  return Object.values(boletas.value).filter((b) => b.status === "reserved")
    .length;
});

const paidTicketsCount = computed(() => {
  return Object.values(boletas.value).filter((b) => b.status === "paid").length;
});

const sortedBoletasForTable = computed(() => {
  const allTickets = [];
  for (let i = 0; i < cantidadBoletas.value; i++) {
    const ticket = boletas.value[i] || {
      status: "available",
      nombre: "",
      telefono: "",
      direccion: "",
      montoPagado: 0,
    };
    allTickets.push({ id: i, ...ticket });
  }
  return allTickets
    .filter((ticket) => ticket.status !== "available")
    .sort((a, b) => a.id - b.id);
});

const SumaMontosPagados = computed(() => {
  return sortedBoletasForTable.value.reduce((total, boleta) => {
    return total + (boleta.montoPagado || 0);
  }, 0);
});

// --- LocalStorage Functions ---
const saveDataToLocalStorage = () => {
  const data = {
    premio: premio.value,
    precioUnitario: precioUnitario.value,
    sorteo: sorteo.value,
    cantidadBoletas: cantidadBoletas.value,
    fechaSorteo: fechaSorteo.value,
    datosGuardados: datosGuardados.value,
    boletas: boletas.value,
    themeColors: themeColors.value,
  };
  localStorage.setItem("raffleAppData", JSON.stringify(data));
};

const loadDataFromLocalStorage = () => {
  const savedData = localStorage.getItem("raffleAppData");
  if (savedData) {
    const data = JSON.parse(savedData);
    premio.value = data.premio || 0;
    premioDisplay.value = new Intl.NumberFormat().format(premio.value);
    precioUnitario.value = data.precioUnitario || 0;
    precioUnitarioDisplay.value = new Intl.NumberFormat().format(
      precioUnitario.value
    );
    sorteo.value = data.sorteo || "";
    cantidadBoletas.value = data.cantidadBoletas || 0;
    fechaSorteo.value = data.fechaSorteo || "";
    datosGuardados.value = data.datosGuardados || false;
    boletas.value = data.boletas || {};
    themeColors.value = data.themeColors || { ...defaultThemeColors };

    if (cantidadBoletas.value > 0) {
      boletaDigits.value = String(cantidadBoletas.value - 1).length;
    }

    if (datosGuardados.value) {
      showWelcomeModal.value = false;
      showDataModal.value = false;
    }
  }
};

// --- Lifecycle Hook: Mounted ---
onMounted(() => {
  loadDataFromLocalStorage();
  if (!datosGuardados.value) {
    showWelcomeModal.value = true;
  }
});

// --- Other Functions ---
const updateBoletas = () => {
  if (cantidadBoletas.value > 0) {
    const newBoletas = {};
    for (let i = 0; i < cantidadBoletas.value; i++) {
      if (boletas.value[i]) {
        newBoletas[i] = boletas.value[i];
      } else {
        newBoletas[i] = {
          status: "available",
          nombre: "",
          telefono: "",
          direccion: "",
          montoPagado: 0,
        };
      }
    }
    boletas.value = newBoletas;
    boletaDigits.value = String(cantidadBoletas.value - 1).length;
  } else {
    boletas.value = {};
    boletaDigits.value = 2;
  }
};

const openDataModal = () => {
  showWelcomeModal.value = false;
  showDataModal.value = true;
};

const editData = () => {
  showDataModal.value = true;
  datosGuardados.value = false;
};

const openAssignTicketModal = (number) => {
  boletaSeleccionada.value = number;
  isEditing.value = false;
  const ticket = boletas.value[number];
  if (ticket) {
    currentTicketData.value = {
      nombre: ticket.nombre,
      telefono: ticket.telefono,
      direccion: ticket.direccion,
      montoPagado: ticket.montoPagado,
      montoPagadoDisplay: new Intl.NumberFormat().format(ticket.montoPagado),
    };
  } else {
    currentTicketData.value = {
      nombre: "",
      telefono: "",
      direccion: "",
      montoPagado: 0,
      montoPagadoDisplay: "",
    };
  }
  showAssignTicketModal.value = true;
};

const closeAssignTicketModal = () => {
  showAssignTicketModal.value = false;
  boletaSeleccionada.value = null;
  isEditing.value = false;
};

const assignTicket = (status) => {
  if (
    !currentTicketData.value.nombre ||
    !currentTicketData.value.telefono ||
    !currentTicketData.value.direccion
  ) {
    Swal.fire({
      icon: "error",
      title: "Error",
      text: "Por favor, ingrese el nombre, teléfono y dirección del participante.",
    });
    return;
  }

  boletas.value[boletaSeleccionada.value] = {
    ...boletas.value[boletaSeleccionada.value],
    status: status,
    nombre: currentTicketData.value.nombre,
    telefono: currentTicketData.value.telefono,
    direccion: currentTicketData.value.direccion,
    montoPagado:
      status === "paid"
        ? precioUnitario.value
        : currentTicketData.value.montoPagado,
  };

  Swal.fire({
    position: "top-end",
    icon: "success",
    title: `Boleta #${boletaSeleccionada.value
      .toString()
      .padStart(boletaDigits.value, "0")} ha sido ${
      status === "reserved" ? "apartada" : "pagada"
    }!`,
    showConfirmButton: false,
    timer: 1500,
  });

  closeAssignTicketModal();
  saveDataToLocalStorage();
};

const markAsPaid = () => {
  boletas.value[boletaSeleccionada.value].status = "paid";
  boletas.value[boletaSeleccionada.value].montoPagado = precioUnitario.value;
  Swal.fire({
    position: "top-end",
    icon: "success",
    title: `Boleta #${boletaSeleccionada.value
      .toString()
      .padStart(boletaDigits.value, "0")} marcada como pagada!`,
    showConfirmButton: false,
    timer: 1500,
  });
  closeAssignTicketModal();
  saveDataToLocalStorage();
};

const releaseTicket = () => {
  boletas.value[boletaSeleccionada.value] = {
    status: "available",
    nombre: "",
    telefono: "",
    direccion: "",
    montoPagado: 0,
  };
  Swal.fire({
    position: "top-end",
    icon: "info",
    title: `Boleta #${boletaSeleccionada.value
      .toString()
      .padStart(boletaDigits.value, "0")} ha sido liberada!`,
    showConfirmButton: false,
    timer: 1500,
  });
  closeAssignTicketModal();
  saveDataToLocalStorage();
};

// Función para habilitar el modo edición
const enableEditMode = () => {
  isEditing.value = true;
  const ticket = boletas.value[boletaSeleccionada.value];
  if (ticket) {
    currentTicketData.value = {
      nombre: ticket.nombre,
      telefono: ticket.telefono,
      direccion: ticket.direccion,
      montoPagado: ticket.montoPagado,
      montoPagadoDisplay: new Intl.NumberFormat().format(ticket.montoPagado),
    };
  }
};

// Función para guardar los cambios editados
const saveEditedData = () => {
  if (
    !currentTicketData.value.nombre ||
    !currentTicketData.value.telefono ||
    !currentTicketData.value.direccion
  ) {
    Swal.fire({
      icon: "error",
      title: "Error",
      text: "Por favor, ingrese el nombre, teléfono y dirección del participante.",
    });
    return;
  }

  boletas.value[boletaSeleccionada.value] = {
    ...boletas.value[boletaSeleccionada.value],
    nombre: currentTicketData.value.nombre,
    telefono: currentTicketData.value.telefono,
    direccion: currentTicketData.value.direccion,
  };

  Swal.fire({
    position: "top-end",
    icon: "success",
    title: "Datos actualizados correctamente!",
    showConfirmButton: false,
    timer: 1500,
  });

  isEditing.value = false;
  saveDataToLocalStorage();
};

function formatCurrency(event, field) {
  let value = event.target.value.replace(/[^0-9]/g, "");
  const numericValue = parseInt(value) || 0;

  if (field === "premio") {
    premio.value = numericValue;
    premioDisplay.value = new Intl.NumberFormat().format(numericValue);
  } else if (field === "precioUnitario") {
    precioUnitario.value = numericValue;
    precioUnitarioDisplay.value = new Intl.NumberFormat().format(numericValue);
  }
  saveDataToLocalStorage();
}

function formatCurrencyForModal(event, field) {
  let value = event.target.value.replace(/[^0-9]/g, "");
  const numericValue = parseInt(value) || 0;

  if (field === "montoPagado") {
    currentTicketData.value.montoPagado = numericValue;
    currentTicketData.value.montoPagadoDisplay = new Intl.NumberFormat().format(
      numericValue
    );
  }
}

function guardarDatos() {
  if (
    premio.value <= 0 ||
    precioUnitario.value <= 0 ||
    !sorteo.value ||
    cantidadBoletas.value <= 0 ||
    !fechaSorteo.value
  ) {
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: "Por favor, complete todos los campos con valores válidos.",
    });
    return;
  }

  datosGuardados.value = true;
  showDataModal.value = false;
  updateBoletas();
  Swal.fire({
    position: "top-end",
    icon: "success",
    title: "¡Datos del sorteo guardados!",
    showConfirmButton: false,
    timer: 1500,
  });
  saveDataToLocalStorage();
}

function formatDate(dateStr) {
  if (!dateStr) return "";
  const options = { year: "numeric", month: "long", day: "numeric" };
  return new Date(dateStr).toLocaleDateString("es-ES", options);
}

function formatDecimal(value) {
  return new Intl.NumberFormat().format(value);
}

const openTicketsTableModal = () => {
  showTicketsTableModal.value = true;
};

const closeTicketsTableModal = () => {
  showTicketsTableModal.value = false;
};

const downloadTicketsPDF = async () => {
  try {
    const { jsPDF } = await import("jspdf");
    const { default: autoTable } = await import("jspdf-autotable");

    const doc = new jsPDF();

    doc.setFontSize(18);
    doc.text("Tabla de Boletas Asignadas", 14, 16);
    doc.setFontSize(12);

    const headers = [
      "# Boleta",
      "Estado",
      "Nombre",
      "Teléfono",
      "Direccion",
      "Monto Pagado",
    ];

    const data = sortedBoletasForTable.value.map((ticket) => [
      `# ${ticket.id}`,
      ticket.status,
      ticket.nombre || "-",
      ticket.telefono || "-",
      ticket.direccion || "-",
      `$ ${formatDecimal(ticket.montoPagado)}`,
    ]);

    autoTable(doc, {
      head: [headers],
      body: data,
      startY: 25,
      theme: "grid",
      headStyles: {
        fillColor: [41, 128, 185],
        textColor: 255,
        fontStyle: "bold",
      },
    });

    const finalY = doc.lastAutoTable.finalY + 10;
    doc.setFont("helvetica", "bold");
    doc.text(`Total:  $ ${formatDecimal(SumaMontosPagados.value)}`, 14, finalY);
    doc.text(`Premio:  $ ${formatDecimal(premio.value)}`, 14, finalY + 7);

    doc.save("boletas_asignadas.pdf");

    Swal.fire({
      position: "top-end",
      icon: "success",
      title: "PDF generado con éxito!",
      showConfirmButton: false,
      timer: 1500,
    });
  } catch (error) {
    console.error("Error al generar PDF:", error);
    Swal.fire({
      icon: "error",
      title: "Error",
      text: "No se pudo generar el PDF: " + error.message,
    });
  }
};

const openSettingsModal = () => {
  showSettingsModal.value = true;
};

const closeSettingsModal = () => {
  showSettingsModal.value = false;
};

const saveThemeSettings = () => {
  saveDataToLocalStorage();
  Swal.fire({
    position: "top-end",
    icon: "success",
    title: "Configuración de tema guardada!",
    showConfirmButton: false,
    timer: 1500,
  });
};

const resetThemeSettings = () => {
  themeColors.value = { ...defaultThemeColors };
  saveDataToLocalStorage();
  Swal.fire({
    position: "top-end",
    icon: "info",
    title: "Tema restablecido a los valores predeterminados!",
    showConfirmButton: false,
    timer: 1500,
  });
};

const showMenu = ref(false);
</script>








<style scoped>

























.app-container {
  font-family: "Arial", sans-serif;
  margin: 0 auto;
  padding: 0px;
  color: #333;
  /* Define CSS variables for theme colors */
  --primary-color: #3498db;
  --secondary-color: #95a5a6;
  --success-color: #2ecc71;
  --danger-color: #e74c3c;
  --header-bg-color: white;
  --header-text-color: #333;
}

/* Update styles to use CSS variables */
.primary-btn,
.btn.primary {
  background-color: var(--primary-color);
  color: white;
}

.secondary-btn,
.btn.secondary {
  background-color: var(--secondary-color);
  color: white;
}

.btn.success {
  background-color: var(--success-color);
  color: white;
}

.btn.danger {
  background-color: var(--danger-color);
  color: white;
}

.btn.info {
  background-color: var(--primary-color);
  /* Using primary for info */
  color: white;
}

.panel-header {
  background-color: var(--header-bg-color);
  color: var(--header-text-color);
  border: 2px solid black;
}

.body-principal {
  display: grid;
  grid-template-columns: 0.5fr 1fr;
}

.badge.active {
  background-color: var(--primary-color);
  color: white;
}

.badge.success {
  background-color: var(--success-color);
  color: white;
}

/* Modal Headers */
.available-modal .modal-header {
  border-top-color: var(--primary-color);
}

.reserved-modal .modal-header {
  border-top-color: var(--danger-color);
}

.paid-modal .modal-header {
  border-top-color: var(--success-color);
}

/* Ticket colors */
.number.available {
  background-color: color-mix(in srgb, var(--primary-color) 20%, white);
  border-color: color-mix(in srgb, var(--primary-color) 50%, white);
  color: color-mix(in srgb, var(--primary-color) 90%, black);
}

.number.reserved {
  background-color: color-mix(in srgb, var(--danger-color) 20%, white);
  border-color: color-mix(in srgb, var(--danger-color) 50%, white);
  color: color-mix(in srgb, var(--danger-color) 90%, black);
}

.number.paid {
  background-color: color-mix(in srgb, var(--success-color) 20%, white);
  border-color: color-mix(in srgb, var(--success-color) 50%, white);
  color: color-mix(in srgb, var(--success-color) 90%, black);
}

/* General Modal Styles (rest of your original modal styles) */
.welcome-modal-container,
.form-modal-container,
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.welcome-modal-content,
.form-modal-content,
.modal-container {
  background: white;
  padding: 25px;
  border-radius: 10px;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
  animation: slideUp 0.3s ease;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 20px;
}

.close-btn {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: #666;
}

/* Estilos para el formulario */
.formulario-sorteo .form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.input-with-icon,
.select-with-icon {
  position: relative;
  display: flex;
  align-items: center;
}

.input-with-icon span,
.select-with-icon span {
  position: absolute;
  left: 10px;
  color: #666;
}

.input-with-icon input,
.select-with-icon select {
  width: 100%;
  padding: 10px 10px 10px 35px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.form-actions {
  margin-top: 20px;
  text-align: right;
}

/* Estilos para los botones */
button {
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  transition: background-color 0.3s;
}

.primary-btn:hover {
  filter: brightness(1.1);
}

.secondary-btn {
  background-color: #f0f0f0;
  color: #333;
}

.secondary-btn:hover {
  background-color: #e0e0e0;
}

/* Estilos para el panel principal */
.main-panel {
  background: white;
  border-radius: 10px;
  overflow: hidden;
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

.badge {
  padding: 3px 8px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: bold;
}

.header-center {
  display: flex;
  justify-content: center;
  flex-grow: 1;
  /* Allow it to take available space */
}

.tab-btn {
  background: none;
  border: none;
  padding: 8px 15px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  gap: 8px;
  color: #7f8c8d;
  cursor: pointer;
}

/* Estilos para las tarjetas de datos */
.data-card {
  border: 5px solid black;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.card-header {
  padding: 10px;
  background: #f8f9fa;
  border-bottom: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 20px;
  font-weight: bold;
}

.card-body {
  padding: 10px;
}

.data-row {
  display: flex;
  justify-content: space-between;
}

.data-label {
  font-weight: bold;
  color: #666;
}

.data-value {
  color: #333;
}

.card-footer {
  padding: 15px;
  text-align: right;
}

/* Modales - Mejoras */
.welcome-modal-container,
.form-modal-container,
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(4px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  animation: fadeIn 0.3s ease;
}

.welcome-modal-content,
.form-modal-content,
.modal-container {
  background: white;
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  overflow: hidden;
  animation: slideUp 0.3s ease;
  padding: 0;
  /* Remove initial padding for better control */
}

/* Modal de bienvenida */
.welcome-modal-content {
  padding: 40px;
  text-align: center;
  max-width: 450px;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

.welcome-modal-content h2 {
  color: #2c3e50;
  font-size: 28px;
  margin-bottom: 10px;
}

.welcome-modal-content p {
  color: #7f8c8d;
  margin-bottom: 30px;
}

.welcome-modal-content button {
  background: linear-gradient(
    to right,
    var(--primary-color),
    var(--success-color)
  );
  color: white;
  border: none;
  padding: 12px 30px;
  border-radius: 50px;
  font-weight: 600;
  display: inline-flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(52, 152, 219, 0.3);
}

.welcome-modal-content button:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 20px rgba(52, 152, 219, 0.4);
}

/* Modal de formulario y boletas */

.form-modal-content {
  max-width: 500px;
  padding: 10px;
}

.modal-container-tabla {
  background-color: white;
  width: 900%;
  max-width: 900px;
}

.modal-container {
  width: 500%;
  max-width: 500px;
}

.form-modal-content .modal-header,
.modal-container .modal-header {
  padding: 20px;
  border-bottom: 1px solid #ecf0f1;
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: relative;
}

.modal-container .modal-header {
  border-top: 8px solid;
  /* For status color */
}

.modal-header h3 {
  color: #2c3e50;
  font-size: 20px;
  margin: 0;
}

.modal-status {
  padding: 3px 10px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: bold;
  text-transform: uppercase;
  color: white;
}

.available-modal .modal-status {
  background-color: var(--primary-color);
}

.reserved-modal .modal-status {
  background-color: var(--danger-color);
}

.paid-modal .modal-status {
  background-color: var(--success-color);
}
#title {
  font-size: 2.5rem;
  font-weight: 700;
  color: #2c3e50;
  line-height: 1.2;
}

.close-btn {
  background: none;
  border: none;
  color: #7f8c8d;
  cursor: pointer;
  font-size: 24px;
  transition: color 0.2s;
}

.close-btn:hover {
  color: #e74c3c;
}

.modal-body {
  padding: 10px;
}

/* Contenido de los modales */
.participant-info,
.payment-info {
  background-color: #f8f9fa;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.info-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 10px;
}

.info-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px dashed #eee;
}

.info-item:last-child {
  border-bottom: none;
}

.info-label {
  font-weight: 500;
  color: #7f8c8d;
}

.info-value {
  color: #2c3e50;
  text-align: right;
}

.payment-info p {
  margin: 8px 0;
  padding-bottom: 8px;
  border-bottom: 1px dashed #ddd;
  display: flex;
  justify-content: space-between;
}

.payment-info p:last-child {
  border-bottom: none;
  padding-bottom: 0;
}

/* Formularios */
.form-container {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.form-group label {
  font-weight: 500;
  color: #555;
}

.input-with-icon,
.select-with-icon {
  position: relative;
}

.input-with-icon span,
.select-with-icon span {
  position: absolute;
  left: 15px;
  top: 50%;
  transform: translateY(-50%);
  color: #7f8c8d;
}

.input-with-icon input,
.select-with-icon select {
  width: 100%;
  padding: 12px 15px 12px 45px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  transition: all 0.3s ease;
}

.input-with-icon input:focus,
.select-with-icon select:focus {
  border-color: var(--primary-color);
  box-shadow: 0 0 0 3px
    color-mix(in srgb, var(--primary-color) 30%, transparent);
  outline: none;
}

.select-with-icon select {
  appearance: none;
  background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3e%3cpolyline points='6 9 12 15 18 9'%3e%3c/polyline%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: right 10px center;
  background-size: 15px;
}

.form-actions,
.action-buttons {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}

/* Botones */
.btn {
  padding: 10px 15px;
  border-radius: 6px;
  border: none;
  font-weight: 500;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  filter: brightness(1.1);
  /* General hover effect for all buttons */
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

@keyframes slideUp {
  from {
    transform: translateY(20px);
    opacity: 0;
  }

  to {
    transform: translateY(0);
    opacity: 1;
  }
}

/* Estadísticas */
.analis {
  border: 5px solid black;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.grid_card {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 15px;
  margin-bottom: 5px;
  padding: 10px;
  /* Added space for the new button */
}

.grid_card .card {
  display: flex;
  background: white;
  padding: 8px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  text-align: center;
  justify-content: center;
}

.grid_card .card h2 {
  font-size: 32px;
  color: black;
  margin-bottom: 5px;
}

.grid_card .card p {
  color: black;
  font-weight: bold;
  font-size: 12px;
}

.information {
  display: flex;
  flex-direction: column;
  gap: 15px;
  padding: 20px;
}

#pagadas {
  background-color: color-mix(in srgb, var(--success-color) 20%, white);
  border-color: color-mix(in srgb, var(--success-color) 50%, white);
  color: color-mix(in srgb, var(--success-color) 90%, black);
}
#apartadas {
  background-color: color-mix(in srgb, var(--danger-color) 20%, white);
  border-color: color-mix(in srgb, var(--danger-color) 50%, white);
  color: color-mix(in srgb, var(--danger-color) 90%, black);
}
#disponible {
  background-color: color-mix(in srgb, var(--primary-color) 20%, white);
  border-color: color-mix(in srgb, var(--primary-color) 50%, white);
  color: color-mix(in srgb, var(--primary-color) 90%, black);
}

.table-button-container {
  text-align: center;
  padding: 10px;
}

/* Estilos para las boletas */
.talonarionumber {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(70px, 1fr));
  gap: 10px;
  padding: 20px;
  justify-items: center;
}

.number {
  width: 60px;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 10px 5px;
  height: 60px;
  /* Fixed height for uniformity */
  border: 1px solid #ddd;
  background: white;
  cursor: pointer;
  transition: all 0.3s;
  border-radius: 8px;
  font-weight: bold;
  font-size: 16px;
  color: #333;
}

.number:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

/* New Table Styles */
.ticket-table-container {
  max-height: 400px;
  overflow-y: auto;
  border: 1px solid #eee;
  border-radius: 8px;
  margin-top: 20px;
}

table {
  width: 100%;
  border-collapse: collapse;
  font-size: 14px;
}

table thead {
  background-color: #f8f9fa;
  position: sticky;
  top: 0;
  z-index: 1;
}

table th,
table td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #eee;
}

table th {
  color: #555;
  font-weight: 600;
  text-transform: uppercase;
}

table tbody tr:hover {
  background-color: #f0f0f0;
}

table tbody tr.reserved {
  background-color: #ffebee;
}

table tbody tr.paid {
  background-color: #e8f5e9;
}

.table-actions {
  text-align: right;
  margin-top: 15px;
}

/* Settings Modal Specific Styles */
.modal-body .form-group {
  margin-bottom: 15px;
}

.modal-body .form-group label {
  font-weight: 500;
  color: #555;
  margin-bottom: 5px;
}

.modal-body input[type="color"] {
  width: 100%;
  height: 40px;
  border: 1px solid #ddd;
  border-radius: 8px;
  cursor: pointer;
  padding: 0;
  /* Remove default padding for color input */
}

/* Hamburguesa */
.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  padding: 10px;
}

.header-left,
.header-center,
.header-right {
  display: flex;
  align-items: center;
  gap: 10px;
}

.badge {
  padding: 4px 8px;
  border-radius: 5px;
  color: white;
}

.hamburger-btn {
  display: none;
  font-size: 24px;
  background: none;
  border: none;
  cursor: pointer;
  color: inherit;
}

.mobile-menu {
  display: none;
  flex-direction: column;
  width: 100%;
  margin-top: 10px;
}

@media (max-width: 1200px) {
  .grid_card {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 15px;
    margin-bottom: 5px;
    /* Added space for the new button */
  }
}

@media (max-width: 700px) {
  .panel-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px 5px;
  }

  .body-principal {
    display: grid;
    grid-template-columns: repeat(1, 1fr);
  }

  .card-body {
    display: flex;
    flex-direction: column;
    padding: 10px;
    gap: 10px;
  }

  .analis {
    padding: 5px;
    border: 5px solid black;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    overflow: hidden;
    margin: 1px;
  }

  .grid_card {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 15px;
    margin-bottom: 5px;
    /* Added space for the new button */
  }

  .talonarionumber {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(50px, 1fr));
    gap: 10px;
    padding: 5px;
  }

  .header-center,
  .header-right {
    display: none;
  }

  .hamburger-btn {
    display: block !important;
  }

  .mobile-menu {
    display: flex;
  }

  .panel-header {
    align-items: flex-start;
  }

  .form-actions,
  .action-buttons {
    display: flex;
    justify-content: flex-end;
    gap: 2px;
    margin-top: 20px;
  }

  .table-actions {
    text-align: center;
    margin-top: 15px;
  }

  .modal-container-tabla[data-v-7a7a37b1] {
    background-color: white;
    width: 900%;
    max-width: 400px;
  }
}

@media (max-width: 430px) {
  .panel-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px 5px;
  }

  .body-principal {
    display: grid;
    grid-template-columns: repeat(1, 1fr);
  }

  .information {
    display: flex;
    flex-direction: column;
    gap: 15px;
    padding: 5px;
  }
  .card-body {
    display: flex;
    flex-direction: column;
    padding: 10px;
    gap: 10px;
  }

  .analis {
    padding: 5px;
    border: 5px solid black;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    overflow: hidden;
    margin: 1px;
  }

  .grid_card {
    display: grid;
    grid-template-columns: repeat(1, 1fr);
    gap: 15px;
    margin-bottom: 20px;
    /* Added space for the new button */
  }

  .talonarionumber {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(50px, 1fr));
    gap: 10px;
    padding: 5px;
  }

  .header-center,
  .header-right {
    display: none;
  }

  .hamburger-btn {
    display: block !important;
  }

  .mobile-menu {
    display: flex;
  }

  .panel-header {
    align-items: flex-start;
  }

  .form-actions,
  .action-buttons {
    display: flex;
    justify-content: flex-end;
    gap: 2px;
    margin-top: 20px;
  }

  .table-actions {
    text-align: center;
    margin-top: 15px;
  }

  .modal-container-tabla[data-v-7a7a37b1] {
    background-color: white;
    width: 900%;
    max-width: 400px;
  }
}
</style>