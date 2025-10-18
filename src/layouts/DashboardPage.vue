<template>
  <div>
    <!-- Sidebar -->
    <aside
      class="bg-dark text-white p-3 position-fixed top-0 start-0 h-100 d-flex flex-column"
      :style="{ width: isCollapsed ? '80px' : '250px', transition: 'width 0.3s' }"
    >
      <!-- Logo + Brand -->
      <div class="d-flex align-items-center mb-4">
        <img
          :src="logo"
          alt="Logo"
          class="rounded-circle"
          width="45"
          height="45"
          role="button"
          @click="isCollapsed ? toggleSidebar() : null"
        />
        <span class="fw-bold ms-2" v-if="!isCollapsed">CPC E-Locker</span>
        <img
          v-if="!isCollapsed"
          :src="brandIcon"
          alt="Hamburger"
          class="ms-auto"
          width="25"
          height="15"
          role="button"
          @click="toggleSidebar"
          title="Collapse"
        />
      </div>

      <!-- Navigation Links -->
      <div class="flex-grow-1 mt-3">
        <div
          v-for="item in navLinks"
          :key="item.label"
          class="nav-link text-white d-flex align-items-center mb-3"
          :class="{ 'justify-content-center': isCollapsed }"
        >
          <RouterLink
            :to="item.path"
            class="d-flex align-items-center text-white text-decoration-none w-100"
            :class="{ 'justify-content-center': isCollapsed }"
            :title="isCollapsed ? item.label : ''"
          >
            <img
              :src="item.img"
              alt="nav icon"
              width="40"
              height="40"
              :class="[isCollapsed ? '' : 'me-2']"
            />
            <span v-if="!isCollapsed">{{ item.label }}</span>
          </RouterLink>
        </div>

        <hr class="border-secondary my-2" />
      </div>

      <!-- Settings -->
      <div
        class="nav-link text-white d-flex align-items-center mb-2"
        :class="{ 'justify-content-center': isCollapsed }"
      >
        <RouterLink
          to="/dashboard/user-settings"
          class="d-flex align-items-center text-white text-decoration-none w-100"
          :class="{ 'justify-content-center': isCollapsed }"
          :title="isCollapsed ? 'Settings' : ''"
        >
          <img
            :src="settingsIcon"
            alt="settings"
            width="40"
            height="40"
            :class="[isCollapsed ? '' : 'me-2']"
          />
          <span v-if="!isCollapsed">Settings</span>
        </RouterLink>
      </div>

      <hr class="border-secondary my-2" />

      <!-- Logout -->
      <div
        class="nav-link text-white d-flex align-items-center"
        :class="{ 'justify-content-center': isCollapsed }"
      >
        <div
          role="button"
          class="d-flex align-items-center text-white text-decoration-none w-100"
          :class="{ 'justify-content-center': isCollapsed }"
          :title="isCollapsed ? 'Logout' : ''"
          @click="logout"
        >
          <img
            :src="logoutIcon"
            alt="logout"
            width="40"
            height="40"
            :class="[isCollapsed ? '' : 'me-2']"
          />
          <span v-if="!isCollapsed">Logout</span>
        </div>
      </div>

      <hr class="border-secondary my-2" />
    </aside>

    <!-- Main Content -->
    <main
      class="p-4 bg-light position-relative"
      :style="{
        marginLeft: isCollapsed ? '80px' : '250px',
        minHeight: '100vh',
        overflowY: 'auto'
      }"
    >
      <!-- 🔔 Notification Bell -->
      <div class="position-absolute top-0 end-0 mt-3 me-4">
        <div class="position-relative">
          <img
            :src="notifIcon"
            alt="Notifications"
            width="35"
            height="35"
            role="button"
            @click="toggleNotifications"
          />
          <span
            v-if="unreadCount > 0"
            class="badge bg-danger position-absolute top-0 start-100 translate-middle rounded-pill"
            style="font-size: 0.7rem"
          >
            {{ unreadCount }}
          </span>
        </div>

        <!-- Notification Dropdown -->
        <div
          v-if="showNotifications"
          class="card shadow-sm position-absolute end-0 mt-2"
          style="width: 320px; z-index: 1050;"
        >
          <div class="card-header d-flex justify-content-between align-items-center">
            <span class="fw-bold">Notifications</span>
            <button class="btn btn-sm btn-outline-primary" @click="markAllRead">Mark all as read</button>
          </div>

          <ul class="list-group list-group-flush">
            <li
              v-for="notif in notifications"
              :key="notif.notif_id"
              class="list-group-item small"
              :class="{ 'bg-light': notif.is_read === 1 }"
              role="button"
              @click="markRead(notif.notif_id)"
            >
              <div>{{ notif.message }}</div>
              <div class="text-muted small">{{ formatDate(notif.created_at) }}</div>
            </li>
            <li v-if="notifications.length === 0" class="list-group-item text-center text-muted">
              No notifications
            </li>
          </ul>
        </div>
      </div>

      <!-- Page Content -->
      <RouterView />
    </main>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import axios from "axios";
import { useLogout } from "@/composables/useLogout";

// ✅ Import Assets
import logo from "@/assets/cpc-logo.jpg";
import brandIcon from "@/assets/hamburger.png";
import lockerIcon from "@/assets/locker.png";
import rentalIcon from "@/assets/rentalstatus.png";
import supportIcon from "@/assets/contactus.png";
import settingsIcon from "@/assets/settings.png";
import logoutIcon from "@/assets/logout.png";
import notifIcon from "@/assets/notification.png"; // Make sure this file exists in /src/assets/

export default {
  setup() {
    const isCollapsed = ref(false);
    const showNotifications = ref(false);
    const notifications = ref([]);
    const unreadCount = ref(0);
    const { logout } = useLogout();

    const API_BASE = "http://localhost:3001"; // ✅ use backend port

    const toggleSidebar = () => (isCollapsed.value = !isCollapsed.value);
    const toggleNotifications = () =>
      (showNotifications.value = !showNotifications.value);

    // ✅ Fetch all notifications for the current user
    const fetchNotifications = async () => {
      try {
        const token = localStorage.getItem("token");
        const { data } = await axios.get(`${API_BASE}/notifications`, {
          headers: { Authorization: `Bearer ${token}` },
        });
        notifications.value = data;
        unreadCount.value = data.filter((n) => n.is_read === 0).length;
      } catch (err) {
        console.error("❌ Error fetching notifications:", err);
      }
    };

    // ✅ Mark a single notification as read
    const markRead = async (notif_id) => {
      try {
        const token = localStorage.getItem("token");
        await axios.put(
          `${API_BASE}/notifications/${notif_id}/read`,
          {},
          { headers: { Authorization: `Bearer ${token}` } }
        );
        await fetchNotifications(); // refresh list
      } catch (err) {
        console.error("❌ Error marking notification as read:", err);
      }
    };

    // ✅ Mark all notifications as read
    const markAllRead = async () => {
      try {
        const token = localStorage.getItem("token");
        await axios.put(
          `${API_BASE}/notifications/read-all`,
          {},
          { headers: { Authorization: `Bearer ${token}` } }
        );
        await fetchNotifications();
      } catch (err) {
        console.error("❌ Error marking all notifications as read:", err);
      }
    };

    // ✅ Format date nicely
    const formatDate = (dateString) => {
      const date = new Date(dateString);
      return date.toLocaleString();
    };

    const navLinks = [
      { label: "Locker", path: "/dashboard/user-locker", img: lockerIcon },
      { label: "Rental Status", path: "/dashboard/user-rental", img: rentalIcon },
      { label: "Support", path: "/dashboard/user-support", img: supportIcon },
    ];

    // ✅ Fetch notifications when component mounts
    onMounted(fetchNotifications);

    return {
      logo,
      brandIcon,
      lockerIcon,
      rentalIcon,
      supportIcon,
      settingsIcon,
      logoutIcon,
      notifIcon,
      isCollapsed,
      showNotifications,
      notifications,
      unreadCount,
      toggleSidebar,
      toggleNotifications,
      markRead,
      markAllRead,
      logout,
      navLinks,
      formatDate,
    };
  },
};
</script>


<style scoped>
main {
  transition: margin-left 0.3s ease;
}
.card {
  max-height: 400px;
  overflow-y: auto;
}
</style>
