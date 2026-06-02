# Central Railway - Diabetic Retinopathy Screening for Mumbai Division (DSCR) 👁️🩺

[![Node.js Version](https://img.shields.io/badge/node-%3E%3D18.20.0-blue.svg)](https://nodejs.org/)
[![Database](https://img.shields.io/badge/Database-PostgreSQL-blue.svg)](https://www.postgresql.org/)
[![License](https://img.shields.io/badge/License-ISC-green.svg)](LICENSE)
[![Docker](https://img.shields.io/badge/Docker-Supported-blue.svg)](Dockerfile)

An end-to-end clinical screening web application designed for the Divisional Railway Hospital, Mumbai Division (Central Railway). It streamlines patient scheduling, clinical recording, and secure data handling for diabetic retinopathy (DR) and macular edema (ME) screenings.

> **Project History & Recognition (Sep 2024 – April 2025)**
>
> Developed as a remote initiative for Central Railway's DSCR health portal, providing a secure, patient-and-doctor-facing health portal.
>
> 🏆 Received a formal **Letter of Appreciation** from the **Chief Medical Superintendent, Divisional Railway Hospital, Mumbai Division, Central Railway**.

---

## Table of Contents

- [What the Project Does](#what-the-project-does)
- [Why the Project is Useful](#why-the-project-is-useful)
- [Technology Stack](#technology-stack)
- [Database Schema](#database-schema)
- [How to Get Started](#how-to-get-started)
  - [Prerequisites](#prerequisites)
  - [Database Setup](#database-setup)
  - [Environment Variables](#environment-variables)
  - [PDF Template Setup](#pdf-template-setup)
  - [Installation & Local Run](#installation--local-run)
  - [Running with Docker](#running-with-docker)
- [Key Features Setup & Integration](#key-features-setup--integration)
  - [WhatsApp Automation](#whatsapp-automation)
  - [Exporting Reports](#exporting-reports)
- [Development Team](#development-team)

---

## What the Project Does

The **Diabetic Retinopathy Portal** provides a secure, structured interface for clinical screening. It captures both patient demographics and detailed ophthalmic/clinical screening measurements over multiple longitudinal visits:

- **Appointment & Booking Flow:** Designed to manage patient registrations and appointments.
- **Patient Demographics:** Capture name, age, sex, contact number, and beneficiary status (employee, retired employee, family).
- **Diabetic Profiling:** Track diabetes type (Type 1, Type 2, MODY), duration, insulin usage, number of Oral Hypoglycemic Agents (OHA), and HbA1c levels.
- **Ophthalmic Parameters:** Record Best-Corrected Visual Acuity (BCVA), Intraocular Pressure (IOP), Diabetic Retinopathy (DR) grading, Macular Edema (ME) staging, and Optical Coherence Tomography (OCT) scan thickness values for both eyes.
- **Ophthalmic Report PDFs:** Dynamically fills a pre-designed clinical PDF template with patient details.
- **Automated WhatsApp Delivery:** Delivers generated PDFs directly to patients' phone numbers.
- **Canvas & Scan Uploads:** Save drawings of fundus structures or upload digital eye scans stored directly as binary data in the database.

---

## Why the Project is Useful

1. **Longitudinal History Tracking:** Instead of siloed records, the portal creates structured logs (`PatientLog`) for each visit, allowing clinicians to review clinical trends.
2. **Automated Patient Communication:** Eliminates manual emailing or printing of reports by sending the PDF automatically to the patient's WhatsApp.
3. **Data Export for Clinical Audits:** Enables researchers and clinicians to export single-patient visit histories or the entire screening database into formatted Excel sheets.
4. **Resilient Architecture:** Runs inside Docker containers with Puppeteer dependencies built-in for headless browser actions (necessary for the WhatsApp client).
5. **Secure Authentication:** Features local secure login with bcrypt password hashing and support for Google OAuth 2.0.

---

## Technology Stack

- **Backend:** Node.js (ES Modules) with [Express.js](https://expressjs.com/)
- **Frontend:** [EJS Templates](https://ejs.co/), [Tailwind CSS](https://tailwindcss.com/) for UI styling
- **Database:** [PostgreSQL](https://www.postgresql.org/) (`pg` pool integration)
- **WhatsApp API:** [whatsapp-web.js](https://wwebjs.dev/) & `@whiskeysockets/baileys`
- **PDF Generation:** [pdf-lib](https://pdf-lib.js.org/) for editing and writing template PDFs
- **Excel Output:** [exceljs](https://github.com/exceljs/exceljs)
- **Authentication:** [Passport.js](http://www.passportjs.org/) (Local and Google OAuth Strategy)

---

## Database Schema

The PostgreSQL schema is structured as follows. Use the SQL statements in [queries.sql](queries.sql) to set up your database.

```mermaid
erDiagram
    DETAILS ||--o{ PATIENTLOG : "has history"
    DETAILS ||--o{ IMAGES : "has scans"
    
    DETAILS {
        text reg PK "Registration Number"
        text name "Patient Name"
        text age
        text sex
        varchar contact
        text beneficiary
        text dtype "Diabetes Type"
        text ddur "Diabetes Duration"
        text insulin
        text oha
        text HBA1c
        text treatment "Array of treatments"
        text bcvar
        text bcval
        text iopr
        text iopl
        text drr
        text drl
        text mer
        text mel
        text octr
        text octl
        text advice "Array of advice"
        text fllwp "Follow up date"
    }

    PATIENTLOG {
        int log_id PK
        int visit "Visit Number"
        text reg FK "Foreign Key to Details"
        text dtype
        text ddur
        text insulin
        text oha
        text HBA1c
        text treatment "Array of treatments"
        text bcvar
        text bcval
        text iopr
        text iopl
        text drr
        text drl
        text mer
        text mel
        text octr
        text octl
        text advice "Array of advice"
        text fllwp
        text notes
        timestamp created_at
    }

    IMAGES {
        int id PK
        text reg FK "Foreign Key to Details"
        varchar filename
        bytea data "Binary Image Blob"
        varchar contenttype
    }

    USERS {
        int id PK
        varchar email "Unique email address"
        varchar password "Hashed Password"
    }
```

---

## How to Get Started

### Prerequisites

- **Node.js**: `v18.x.x` (Recommended: `18.20.8`)
- **PostgreSQL**: `v14+` running locally or hosted online.
- **Google Developer Credentials** (optional, for Google Sign-in)

### Database Setup

1. Log into your PostgreSQL instance.
2. Create a new database:
   ```sql
   CREATE DATABASE dscr_db;
   ```
3. Run the schema creation script from the root project directory:
   ```bash
   psql -d dscr_db -f queries.sql
   ```

### Environment Variables

Create a `.env` file in the root directory of the project:

```env
PORT=3000
UNAME=dscr_admin # Any non-empty string to verify env load
DB_URL=postgres://postgres:123456@localhost:5432/dscr_db
NODE_ENV=development # Set to production to enforce secure DB SSL connection
DB_CA_CERT= # Optional CA certificate for database connection in production
```

### PDF Template Setup

In order to generate PDF screening forms, you must provide a template PDF file:
1. Ensure the directory path `public/templates/` exists.
2. Put your screening PDF named **`DM screening Form.pdf`** inside the `public/templates/` directory. The application matches exact coordinates on this template to overlay patient information.

### Installation & Local Run

1. Install project dependencies:
   ```bash
   npm install
   ```
2. Start the Express server:
   ```bash
   npm start
   ```
3. The server will run at: `http://localhost:3000/home`

### Running with Docker

Since the app requires Chrome dependencies to run the WhatsApp client (via Puppeteer), deploying via Docker is highly recommended to avoid local binary issues.

1. Build the Docker image:
   ```bash
   docker build -t dscr-portal .
   ```
2. Run the container:
   ```bash
   docker run -p 3000:3000 -e DB_URL=your_postgres_db_url dscr-portal
   ```

---

## Key Features Setup & Integration

### WhatsApp Automation

The application uses `whatsapp-web.js` under the hood. When the app starts:
1. An authentication QR code is generated and logged directly in the terminal as an ASCII QR code.
2. A REST API route at `/get-qr-code` serves this QR code as a data URL for display in your custom login/dashboard pages.
3. Scan this QR code using the WhatsApp app on your smartphone (*Linked Devices* -> *Link a Device*).
4. Once authenticated, a persistent session is stored in the `./sessions` directory so you won't need to re-scan on every server restart.

### Exporting Reports

- **All Data Export**: Hit `GET /export-to-excel-all` to download a complete spreadsheet of all patient logs.
- **Individual Patient Data**: Hit `GET /export-to-excel/:id` where `:id` is the patient's registration number to fetch a detailed report for that specific patient.

## 👥 Development Team

This portal was designed, built, and delivered by:

<div align="center">
  <table>
    <tr>
      <td align="center" width="180">
        <a href="https://github.com/dhruvp-dev">
          <img src="https://github.com/dhruvp-dev.png" width="80" style="border-radius: 50%;" alt="Dhruv P"/><br />
          <sub><b>Dhruv Pandey</b></sub>
        </a><br />
        <small>Backend Developer</small>
      </td>
      <td align="center" width="180">
        <a href="https://github.com/ramanan-2735">
          <img src="https://github.com/ramanan-2735.png" width="80" style="border-radius: 50%;" alt="Ramanan"/><br />
          <sub><b>Ramanan</b></sub>
        </a><br />
        <small>Full Stack Developer</small>
      </td>
      <td align="center" width="180">
        <a href="https://github.com/anshvermadev">
          <img src="https://github.com/anshvermadev.png" width="80" style="border-radius: 50%;" alt="Ansh Verma"/><br />
          <sub><b>Ansh Verma</b></sub>
        </a><br />
        <small>Frontend Developer</small>
      </td>
    </tr>
  </table>
</div>
