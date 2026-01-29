# 💼 Job Application Dashboard

A modern, responsive **CRUD (Create, Read, Update, Delete)** application built to help developers track their job hunt progress. 

This project demonstrates the power of **Vue 3 (Composition API)** combined with **Vuetify** for a professional Material Design interface.

## 🚀 Features

* **Create:** Add new job applications with Company, Role, Status, and Date.
* **Read:** View all applications in a sortable, paginated Data Table.
* **Update:** Edit existing applications using a modal popup.
* **Delete:** Remove applications with instant UI updates.
* **Visual Status:** Color-coded chips for different application statuses (Applied, Interviewing, Rejected).
* **Responsive:** Fully mobile-friendly interface using Vuetify's Grid system.

## 🛠️ Tech Stack

* **Framework:** [Vue.js 3](https://vuejs.org/) (Script Setup / Composition API)
* **UI Library:** [Vuetify 3](https://vuetifyjs.com/) (Material Design)
* **Build Tool:** [Vite](https://vitejs.dev/)
* **Icons:** Material Design Icons (MDI)

## 📸 Usage

### The Dashboard
The main view utilizes `v-data-table` to organize complex data.
*(You can upload a screenshot here later)*

### Adding/Editing Jobs
Uses a reusable `v-dialog` component with form validation to handle both creating new entries and editing existing ones.

## 💻 Project Setup

If you want to run this project locally:

```bash
# 1. Clone the repository
git clone [https://github.com/YOUR_USERNAME/job-application-dashboard.git](https://github.com/YOUR_USERNAME/job-application-dashboard.git)

# 2. Enter the folder
cd job-application-dashboard

# 3. Install dependencies
npm install

# 4. Run the development server
npm run dev
