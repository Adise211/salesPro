<template>
  <div class="default-layout">
    <v-layout class="rounded rounded-md">
      <v-navigation-drawer color="blue-grey-darken-4" persistent>
        <v-list>
          <v-list-item
            class="py-5"
            :title="userFullName"
            :subtitle="userEmail"
            prepend-avatar="https://randomuser.me/api/portraits/women/85.jpg"
          ></v-list-item>
          <v-divider></v-divider>
          <v-list-item
            v-for="(item, index) in navItems"
            :key="index"
            :value="item.value"
            link
            :title="item.title"
            :append-icon="item.icon"
            @click="onNavItemClick(item.value)"
          ></v-list-item>
        </v-list>

        <template v-slot:append>
          <div class="pa-2">
            <v-btn block color="primary" @click="onLogout"> Logout </v-btn>
          </div>
        </template>
      </v-navigation-drawer>
      <v-app-bar color="blue-grey-darken-4" flat title="SalesPro"> </v-app-bar>

      <v-main class="main-content">
        <slot></slot>
      </v-main>
    </v-layout>
    <!-- Reminder Alert Dialog -->
    <v-dialog v-if="currentNote" v-model="isReminderDialogOpen" width="500">
      <v-card>
        <!-- <v-toolbar color="#eab308" class="px-2 text-h5 text-white" height="56" elevation="3"
          >Reminder</v-toolbar
        > -->
        <v-card-title class="text-h5 font-weight-medium">Reminder</v-card-title>
        <v-card-text>
          <div>
            <div>
              Reffer to:
              <span class="text-medium-emphasis">{{ currentNote.CompanyName }}</span>
            </div>
            <div>
              Date: <span class="text-medium-emphasis">{{ currentNote.SelectedDate }}</span>
            </div>
            <div>
              Time: <span class="text-medium-emphasis">{{ currentNote.SelectedTime }}</span>
            </div>
            <div class="text-center">{{ currentNote.NoteDescription }}</div>
          </div>
        </v-card-text>
        <v-card-actions class="d-flex justify-center align-center">
          <v-btn variant="flat" color="primary" @click="closeDialog">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import { NavigationItems, SaleStatuses } from "@/utilities/consts";
import { mapState, mapActions } from "pinia";
import { useGeneralStore } from "@/stores/general";
import { useCalendarStore } from "@/stores/calendar";
import { logoutUser } from "@/firebase/services/user";
import { updateNoteWatchedTime } from "@/firebase/services/data";
import Config from "@/utilities/config";
import { convertTime } from "@/utilities/utilsFuncs";

let checkUserActivityInterval;

export default {
  name: "DefaultLayout",
  components: {},
  props: {
    displayNote: {
      type: Object,
      default: () => null
    }
  },
  data: () => ({
    isReminderDialogOpen: false,
    currentNote: null
  }),
  created() {},
  mounted() {
    // Check if user is still active (interval)
    checkUserActivityInterval = setInterval(
      this.sessionExparationHandler(),
      convertTime(Config.session.userActivityCheckInterval).miliseconds
    );
  },
  unmounted() {
    clearInterval(checkUserActivityInterval);
  },
  methods: {
    ...mapActions(useGeneralStore, ["updateUserNotesListInStore"]),
    onNavItemClick(itemName) {
      let pageName;
      let paramsObj;

      switch (itemName) {
        case NavigationItems.OverView:
          pageName = "OverViewPage";
          paramsObj = {};
          break;
        case NavigationItems.Calendar:
          pageName = "CalendarPage";
          paramsObj = {};
          break;
        case NavigationItems.Notes:
          pageName = "NotesPage";
          paramsObj = {};
          break;
        case NavigationItems.Sales:
          pageName = "SalesPage";
          paramsObj = { stageName: SaleStatuses.FollowUps.value };
          break;
        case NavigationItems.Settings:
          pageName = "UserSettingsPage";
          paramsObj = {};
          break;
        default:
          break;
      }
      this.$router.push({
        name: pageName,
        params: paramsObj
      });
    },
    async onLogout() {
      const response = await logoutUser();
      if (response?.isUserLogedout) {
        // Reset store (user info)
        useGeneralStore().$reset();
        useCalendarStore().$reset();
        // Redirect to login page
        this.$router.push({
          name: "LoginPage"
        });
      }
    },
    sessionExparationHandler() {
      if (!this.isSessionUserActive) {
        console.log("logged out auto!!!");
        this.onLogout();
      }
    },
    async closeDialog() {
      this.isReminderDialogOpen = false;
      if (this.currentNote.NoteId === this.noteReminder?.NoteId) {
        const response = await updateNoteWatchedTime(this.noteReminder);
        if (response.success) {
          this.updateUserNotesListInStore(response.data);
        }
      }
      this.currentNote = null;
    }
  },
  computed: {
    ...mapState(useGeneralStore, [
      "userEmail",
      "userFullName",
      "isSessionUserActive",
      "noteReminder"
    ]),
    navItems() {
      return [
        {
          title: "Overview",
          value: "Overview",
          icon: "mdi-chart-bar",
          isShownNow: false
        },
        {
          title: "Calendar",
          value: "Calendar",
          icon: "mdi-calendar",
          isShownNow: false
        },
        {
          title: "Notes",
          value: "Notes",
          icon: "mdi-note-text",
          isShownNow: false
        },
        {
          title: "Sales",
          value: "Sales",
          icon: "mdi-sale",
          isShownNow: false
        },
        {
          title: "Extract From Files",
          value: "ExtractFiles",
          icon: "mdi-file-cog",
          isShownNow: false
        },
        {
          title: "Settings",
          value: "Settings",
          icon: "mdi-cog",
          isShownNow: false
        }
      ];
    }
  },
  watch: {
    noteReminder: {
      handler(newVal) {
        if (newVal) {
          this.currentNote = newVal;
          // If there is a new reminder for popup - open reminder with delay
          setTimeout(() => {
            this.isReminderDialogOpen = true;
          }, 5000);
        }
      },
      immediate: true
    },
    displayNote: {
      handler(newVal) {
        this.currentNote = newVal;
        this.isReminderDialogOpen = true;
      },
      immediate: true
    }
  }
};
</script>

<style scoped></style>
