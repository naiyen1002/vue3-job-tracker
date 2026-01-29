<script setup>
import { ref, onMounted, watch } from "vue";

// --- 1. TABLE CONFIGURATION ---
// Defines the columns for the v-data-table component.
// 'key' matches the property name in our data objects.
const headers = [
  { title: "Company Name", align: "start", key: "company" },
  { title: "Job Role", align: "start", key: "role" },
  { title: "Application Date", align: "end", key: "date" },
  { title: "Current Status", align: "center", key: "status" },
  { title: "Link", key: "url", sortable: false, align: "center" },
  { title: "Actions", key: "actions", sortable: false },
];

// --- 2. MAIN DATA STATE ---
// 'ref' makes this array reactive. If we push/remove items, the UI updates automatically.
const applications = ref([
  {
    id: 1,
    company: "Google",
    role: "Frontend Developer",
    status: "Interviewing",
    url: "https://google.com/careers",
    date: "2026-01-20",
  },
  {
    id: 2,
    company: "Tesla",
    role: "Vue Engineer",
    status: "Applied",
    url: "",
    date: "2026-01-25",
  },
  {
    id: 3,
    company: "Amazon",
    role: "Junior Dev",
    status: "Rejected",
    url: "",
    date: "2026-01-15",
  },
]);

// --- 3. FORM STATE ---
// This object holds the data currently being typed into the "Add/Edit" popup.
const newJob = ref({
  company: "",
  role: "",
  date: null,
  status: "Applied",
});

// Controls whether the popup dialog is visible (true) or hidden (false).
const dialog = ref(false);

// --- STATE: TRACKING EDIT MODE ---
// editedIndex is used to track which item is being edited.
// -1 means we are in "Add Mode" (creating a new job).
// 0 or higher means we are in "Edit Mode" (updating an existing job at that index).
const editedIndex = ref(-1);

// Inside script setup
const rules = [
  (value) => !!value || "Required.",
  (value) => (value && value.length >= 3) || "Min 3 characters",
];

const search = ref("");

const snackbar = ref(false);
const snackbarText = ref("");
// --- 4. ACTIONS / FUNCTIONS ---

// Called when the "Pencil" icon is clicked.
function editJob(item) {
  // Fix for Vuetify 3 wrapper (get the clean data object)
  const itemData = item.raw || item;

  // Set editedIndex to the index of the item we clicked.
  // This switches the form from "Add Mode" to "Edit Mode".
  editedIndex.value = applications.value.indexOf(itemData);

  // Copy the item's data into the form model (newJob).
  // We use Object.assign({}) to create a copy, so we don't modify the table row directly while typing.
  newJob.value = Object.assign({}, itemData);

  // Open the popup
  dialog.value = true;
}

// Called when the "Save" button is clicked.
// Handles both creating new items and updating existing ones.
function save() {
  if (!newJob.value.company || !newJob.value.role) return;

  // Check if we are editing an existing item or creating a new one
  if (editedIndex.value > -1) {
    // --- CASE 1: EDITING ---
    // We found an index, so we update the item at that specific position in the array.
    Object.assign(applications.value[editedIndex.value], newJob.value);
  } else {
    // --- CASE 2: CREATING ---
    // editedIndex is -1, so we create a new object and push it to the list.
    applications.value.push({
      id: Date.now(),
      company: newJob.value.company,
      role: newJob.value.role,
      status: newJob.value.status,
      url: newJob.value.url,
      date: newJob.value.date,
    });
    
  }
  
  snackbarText.value = "Application Saved Successfully!";
  snackbar.value = true;

  close();
}

// Helper to close dialog and reset state
function close() {
  dialog.value = false;
  // Wait for the dialog close animation (300ms) to finish before resetting the form.
  setTimeout(() => {
    // Reset form fields to empty and set editedIndex back to -1 (Add Mode)
    newJob.value = { company: "", role: "", date: null, status: "Applied" };
    editedIndex.value = -1;
  }, 300);
}

// Called when the "Trash" icon is clicked.
function deleteJob(itemToDelete) {
  // Get the ID of the item to delete
  const idToDelete = itemToDelete.raw ? itemToDelete.raw.id : itemToDelete.id;
  // Filter the array: Keep only items that DO NOT match the ID we want to delete.
  applications.value = applications.value.filter(
    (job) => job.id !== idToDelete,
  );
}

function getStatusColor(status) {
  if (status === "Rejected") return "red";
  if (status === "Interviewing") return "orange";
  if (status === "Offer") return "green";
  if (status === "Applied") return "blue";
  return "grey"; // Default color
}

onMounted(() => {
  const savedData = localStorage.getItem("my-job-apps");
  if (savedData) {
    applications.value = JSON.parse(savedData);
  }
});

// Watch for ANY change in 'applications' and save it immediately
watch(
  applications,
  (newVal) => {
    localStorage.setItem("my-job-apps", JSON.stringify(newVal));
  },
  { deep: true },
); // 'deep' is crucial for arrays/objects!
</script>

<template>
  <!-- v-app is the root component required for Vuetify to work correctly -->
  <v-app>
    <v-main>
      <v-container>
        <v-row class="mb-4" align="center">
          <v-col>
            <h1 class="text-h4">Job Hunt Dashboard</h1>
          </v-col>
          <v-col class="searchbar">
            <v-text-field
              v-model="search"
              label="Search"
              prepend-inner-icon="mdi-magnify"
            />
          </v-col>
          <v-col class="text-right">
            <v-btn
              color="primary"
              prepend-icon="mdi-plus"
              @click="dialog = true"
            >
              New Application
            </v-btn>
          </v-col>
        </v-row>

        <!-- Popup Dialog (Modal) -->
        <!-- v-model="dialog" controls visibility based on the 'dialog' ref -->
        <v-dialog v-model="dialog" max-width="500px">
          <v-card>
            <v-card-title>
              {{ editedIndex === -1 ? "Add New Job" : "Edit Job Details" }}
            </v-card-title>

            <v-card-text>
              <!-- Form Fields linked to 'newJob' data via v-model -->
              <v-text-field
                label="Company Name"
                v-model="newJob.company"
                variant="outlined"
                class="mb-3"
                :rules="rules"
              ></v-text-field>

              <v-text-field
                label="Job Role"
                v-model="newJob.role"
                variant="outlined"
                class="mb-3"
                :rules="rules"
              ></v-text-field>

              <v-select
                label="Status"
                v-model="newJob.status"
                :items="['Applied', 'Interviewing', 'Rejected', 'Offer']"
                variant="outlined"
                class="mb-3"
              ></v-select>

              <v-text-field
                label="Application Date"
                v-model="newJob.date"
                type="date"
                variant="outlined"
                class="mb-3"
              ></v-text-field>

              <v-text-field
                label="Link"
                v-model="newJob.url"
                variant="outlined"
                class="mb-3"
              ></v-text-field>
            </v-card-text>

            <v-card-actions>
              <v-btn color="red" @click="close">Cancel</v-btn>
              <v-btn color="primary" @click="save">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <!-- Data Table -->
        <!-- :headers defines columns, :items defines the data rows, :search allows filtering-->
        <v-data-table
          :headers="headers"
          :items="applications"
          :search="search"
          :loading="loading"
          class="elevation-1"
        >
          <!-- Custom Slot for 'status' column to show colored chips -->
          <template v-slot:item.status="{ item }">
            <v-chip
              :color="getStatusColor(item.raw ? item.raw.status : item.status)"
            >
              {{ item.raw ? item.raw.status : item.status }}
            </v-chip>
          </template>

          <template v-slot:item.url="{ item }">
            <v-btn
              v-if="item.raw ? item.raw.url : item.url"
              icon="mdi-open-in-new"
              size="small"
              color="primary"
              variant="text"
              :href="item.raw ? item.raw.url : item.url"
              target="_blank"
            ></v-btn>
          </template>

          <!-- Custom Slot for 'actions' column to show Edit/Delete buttons -->
          <template v-slot:item.actions="{ item }">
            <v-btn
              icon="mdi-pencil"
              size="small"
              color="blue"
              variant="text"
              class="mr-2"
              @click="editJob(item)"
            ></v-btn>

            <v-btn
              icon="mdi-delete"
              size="small"
              color="red"
              variant="text"
              @click="deleteJob(item)"
            ></v-btn>
          </template>
        </v-data-table>
      </v-container>
    </v-main>
  </v-app>

  <v-snackbar v-model="snackbar">{{ snackbarText }}</v-snackbar>
</template>
