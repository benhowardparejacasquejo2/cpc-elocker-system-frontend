<template>
  <div class="container py-4">
    <h2 class="text-center fw-bold mb-4 text-primary">RENTAL STATUS</h2>

    <div class="row justify-content-center">
      <div
        v-for="(rental, index) in rentals"
        :key="index"
        class="col-md-6 col-lg-5 d-flex justify-content-center"
      >
        <div class="card rental-card shadow-bottom border-0 m-3">
          <div class="card-body d-flex justify-content-between align-items-center text-center">
            <!-- LEFT SIDE -->
            <div class="left-section w-50">
              <div class="d-flex flex-column align-items-center mb-3">
                <img
                  src="@/assets/user.png"
                  alt="Profile"
                  class="rounded-circle border mb-2"
                  width="70"
                  height="70"
                />
              </div>

              <div>
                <p class="mb-1"><strong>Student ID:</strong> {{ rental.stud_id }}</p>
                <p class="mb-1">
                  <strong>Student Name:</strong> {{ rental.f_name }} {{ rental.l_name }}
                </p>
                <p class="mb-1"><strong>Course:</strong> {{ rental.course_name }}</p>
                <p class="mb-1"><strong>Email:</strong> {{ rental.email }}</p>
              </div>
            </div>

            <!-- RIGHT SIDE -->
            <div class="right-section w-50">
              <div class="mb-2">
                <i class="bi bi-hdd-stack fs-3 text-secondary"></i>
                <div class="fw-bold">{{ rental.locker_number }}</div>
              </div>

              <div>
                <p class="mb-1">
                  <strong>Rental Start:</strong><br />{{ formatDate(rental.start_date) }}
                </p>
                <p class="mb-3">
                  <strong>Rental End:</strong><br />{{ formatDate(rental.due_date) }}
                </p>
              </div>

              <div class="d-flex flex-column align-items-center gap-2">
                <button
                  class="btn btn-primary btn-sm"
                  @click="openPaymentHistory(rental)"
                >
                  Payment History
                </button>

                <button
                  class="btn btn-outline-primary btn-sm"
                  @click="openRenewModal(rental)"
                >
                  Renew Locker
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- NO DATA -->
      <div v-if="rentals.length === 0" class="text-muted text-center mt-5">
        <i class="bi bi-info-circle"></i> No rental records found
      </div>
    </div>

    <!-- 🧾 Payment History Modal -->
    <div
      class="modal fade"
      id="paymentModal"
      tabindex="-1"
      aria-labelledby="paymentModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title fw-bold" id="paymentModalLabel">Payment History</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
          </div>
          <div class="modal-body">
            <div v-if="paymentHistory">
              <p><strong>Status:</strong> {{ paymentHistory.status }}</p>
              <p><strong>Start Date:</strong> {{ formatDate(paymentHistory.start_date) }}</p>
              <p><strong>Due Date:</strong> {{ formatDate(paymentHistory.due_date) }}</p>
              <p><strong>Payment Method:</strong> {{ paymentHistory.payment_method }}</p>
              <p><strong>Balance:</strong> ₱{{ paymentHistory.balance }}</p>
              <p><strong>Paid Amount:</strong> ₱{{ paymentHistory.paid_amount }}</p>
              <p><strong>Total Amount:</strong> ₱{{ paymentHistory.total_amount }}</p>
            </div>
            <div v-else class="text-center text-muted">
              <i class="bi bi-clock"></i> Loading payment history...
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 🔁 Renew Locker Modal -->
    <div
      class="modal fade"
      id="renewModal"
      tabindex="-1"
      aria-labelledby="renewModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title fw-bold" id="renewModalLabel">
              Renew Locker {{ renewForm.lockerNumber }}
            </h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
          </div>

          <div class="modal-body">
            <form @submit.prevent="submitRenewal">
              <div class="mb-3">
                <label class="form-label fw-semibold">Student ID</label>
                <input v-model="renewForm.studentId" type="text" class="form-control" required />
              </div>

              <div class="mb-3">
                <label class="form-label fw-semibold">Student Name</label>
                <input v-model="renewForm.studentName" type="text" class="form-control" required />
              </div>

              <div class="mb-3">
                <label class="form-label fw-semibold">Course</label>
                <input v-model="renewForm.course" type="text" class="form-control" required />
              </div>

              <div class="mb-3">
                <label class="form-label fw-semibold">Email</label>
                <input v-model="renewForm.email" type="email" class="form-control" required />
              </div>

              <div class="mb-3">
                <label class="form-label fw-semibold">New Rental Start</label>
                <input v-model="renewForm.rentalStart" type="date" class="form-control" required />
              </div>

              <div class="mb-3">
                <label class="form-label fw-semibold">New Rental End</label>
                <input v-model="renewForm.rentalEnd" type="date" class="form-control" required />
              </div>

              <div class="d-flex justify-content-end gap-2">
                <button type="button" class="btn btn-danger" data-bs-dismiss="modal">Cancel</button>
                <button type="submit" class="btn btn-primary">Renew</button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// eslint-disable-next-line
import * as bootstrap from "bootstrap";
import axios from "axios";

export default {
  name: "RentStatus",
  data() {
    return {
      rentals: [],
      paymentHistory: null,
      paymentModal: null,
      renewModal: null,
      renewForm: {
        rentalId: "",
        lockerNumber: "",
        studentId: "",
        studentName: "",
        course: "",
        email: "",
        rentalStart: "",
        rentalEnd: "",
      },
    };
  },
  methods: {
    async fetchRentals() {
      try {
        const res = await axios.get("http://localhost:3001/rent-status");
        this.rentals = res.data;
      } catch (err) {
        console.error("Error fetching rent status:", err);
      }
    },

    async openPaymentHistory(rental) {
      this.paymentHistory = null;
      try {
        const res = await axios.get(
          `http://localhost:3001/payment-history/${rental.rental_id}`,
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          }
        );
        this.paymentHistory = res.data;
      } catch (err) {
        console.error("Error fetching payment history:", err);
      }

      if (!this.paymentModal) {
        const modalEl = document.getElementById("paymentModal");
        this.paymentModal = new bootstrap.Modal(modalEl);
      }
      this.paymentModal.show();
    },

    openRenewModal(rental) {
      this.renewForm = {
        rentalId: rental.rental_id,
        lockerNumber: rental.locker_number,
        studentId: rental.stud_id,
        studentName: `${rental.f_name} ${rental.l_name}`,
        course: rental.course_name,
        email: rental.email,
        rentalStart: new Date().toISOString().slice(0, 10),
        rentalEnd: "",
      };

      if (!this.renewModal) {
        const modalEl = document.getElementById("renewModal");
        this.renewModal = new bootstrap.Modal(modalEl);
      }
      this.renewModal.show();
    },

    async submitRenewal() {
      try {
        await axios.post("http://localhost:3001/client/renew", this.renewForm);
        alert("✅ Locker renewed successfully!");
        this.renewModal.hide();
        this.fetchRentals();
      } catch (error) {
        console.error("Error renewing locker:", error);
        alert("❌ Failed to renew locker. Please try again.");
      }
    },

    formatDate(dateStr) {
      if (!dateStr) return "—";
      const date = new Date(dateStr);
      return date.toLocaleDateString("en-US", {
        year: "numeric",
        month: "long",
        day: "numeric",
      });
    },
  },
  mounted() {
    this.fetchRentals();
  },
};
</script>

<style scoped>
.rental-card {
  width: 550px;
  border-radius: 14px;
  background-color: #fff;
  padding: 18px 20px;
  margin: 15px;
}

.shadow-bottom {
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.12);
}

.card-body {
  font-size: 14px;
  color: #333;
}

.left-section,
.right-section {
  text-align: center;
}

.btn {
  border-radius: 20px;
  font-weight: 500;
  padding: 6px 18px;
}

.btn-primary {
  background-color: #007bff;
  border: none;
}

.btn-outline-primary {
  border: 1px solid #007bff;
  color: #007bff;
}

.btn-outline-primary:hover {
  background-color: #007bff;
  color: #fff;
}

img {
  object-fit: cover;
  border: 2px solid #ddd;
}

.rental-card:hover {
  transform: translateY(-4px);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  box-shadow: 0 10px 18px rgba(0, 0, 0, 0.18);
}
</style>
