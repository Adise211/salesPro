<template>
  <v-container fluid class="fill-height py-0">
    <v-row class="fill-height">
      <v-col md="8">
        <AppCard>
          <template v-slot:card-text>
            <v-row class="mb-7 pt-2 align-center">
              <v-col md="5">
                <v-text-field
                  v-model="searchExpression"
                  hide-details
                  single-line
                  placeholder="Search here..."
                  append-inner-icon="mdi-magnify"
                >
                </v-text-field>
              </v-col>
              <v-spacer></v-spacer>
              <v-col md="4" class="text-end">
                <v-btn variant="text" color="primary" prepend-icon="mdi-plus">Add Note</v-btn>
              </v-col>
            </v-row>
            <v-data-table
              class="notes-table"
              :headers="tableHeaders"
              :items="tableItems"
              :items-per-page="itemsPerPage"
              :height="415"
              :page="page"
              :search="searchExpression"
            >
              <template v-slot:item="{ item }">
                <tr>
                  <td class="text-medium-emphasis">{{ item.NoteId }}</td>
                  <td class="text-medium-emphasis">{{ item.CompanyName }}</td>
                  <td class="text-medium-emphasis">{{ item.SelectedDate }}</td>
                  <td class="text-center">
                    <v-icon v-if="item.RemindMe" icon="mdi-bell" color="#eab308"></v-icon>
                  </td>
                  <td class="text-center">
                    <v-icon icon="mdi-pencil" @click="onEditNoteClick(item)"></v-icon>
                  </td>
                  <td class="text-center">
                    <v-icon @click="onDeleteIconClick(item)" color="error"> mdi-delete </v-icon>
                  </td>
                </tr>
              </template>
              <template v-slot:bottom>
                <div class="text-center pt-2">
                  <v-pagination v-model="page" :length="pageCount"></v-pagination>
                </div>
              </template>
            </v-data-table>
          </template>
        </AppCard>
      </v-col>
      <v-col md="4">
        <div class="d-flex flex-column fill-height">
          <AppCard class="view-notes h-50">
            <template v-slot:card-text>
              <div>View Note (by id)</div>
            </template>
          </AppCard>
          <AppCard class="create-notes mt-3" :cardTextClass="'fill-height'">
            <template v-slot:card-text>
              <div class="fill-height d-flex flex-column">
                <v-form ref="newNoteForm">
                  <!-- Reffer To -->
                  <v-autocomplete
                    v-model="companyName"
                    :items="companiesList"
                    item-title="Company"
                    item-value="Company"
                    :rules="[formRules.required]"
                    placeholder="Reffer to"
                    variant="outlined"
                    density="compact"
                    color="primary"
                    style="max-width: 50%"
                  >
                  </v-autocomplete>
                  <div class="d-flex">
                    <!-- Date -->
                    <v-text-field
                      v-model="selectedDate"
                      type="date"
                      :rules="remindMe ? [formRules.required] : []"
                      variant="outlined"
                      density="compact"
                      color="primary"
                      style="max-width: 30%"
                    >
                    </v-text-field>
                    <!-- Time -->
                    <v-text-field
                      v-model="selectedTime"
                      type="time"
                      :rules="remindMe ? [formRules.required] : []"
                      class="ml-3"
                      variant="outlined"
                      density="compact"
                      color="primary"
                      style="max-width: 20%"
                    >
                    </v-text-field>
                  </div>
                  <!-- Note Description -->
                  <v-textarea
                    v-model="noteDescription"
                    :rules="remindMe ? [formRules.required] : []"
                    placeholder="Type here..."
                    variant="outlined"
                    color="primary"
                    rows="3"
                  >
                  </v-textarea>
                  <!-- Remind me checkbox (activate reminder) -->
                  <v-checkbox
                    label="Remind me"
                    v-model="remindMe"
                    color="#eab308"
                    density="compact"
                  ></v-checkbox>
                </v-form>
                <!-- Action Buttons -->
                <div class="mt-auto mb-3">
                  <v-btn class="mr-2" color="primary" @click="saveNote" :loading="isSaveNoteLoading"
                    >Save</v-btn
                  >
                  <v-btn variant="outlined" color="primary" @click="onClearForm">Clear</v-btn>
                </div>
              </div>
            </template>
          </AppCard>
        </div>
      </v-col>
    </v-row>
    <!-- Before Delete Dialog -->
    <v-dialog v-model="isDeleteDialogOpen" width="500">
      <v-card height="120px">
        <v-card-text>Are you sure you want to delete this item?</v-card-text>
        <v-card-actions>
          <div class="d-flex flex-row justify-start ml-auto">
            <v-btn color="primary" variant="flat" @click="deleteItem(currentItem)">I am sure</v-btn>
            <v-btn color="primary" variant="outlined" @click="isDeleteDialogOpen = false"
              >Never mind</v-btn
            >
          </div>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <!-- Create/Update Note -->
    <AppNoteDialog
      :isDialogOpen="isNoteDialogOpen"
      @onDialogClose="isNoteDialogOpen = false"
    ></AppNoteDialog>
  </v-container>
</template>

<script>
import AppCard from "@/components/AppCard.vue";
import AppNoteDialog from "@/components/AppNoteDialog.vue";
import { mapState, mapActions } from "pinia";
import { useGeneralStore } from "@/stores/general";
import { createNewNote, updateNote, removeNote } from "@/firebase/services/data";
import { ToastMessages } from "@/utilities/consts";

export default {
  name: "NotesPage",
  components: { AppCard, AppNoteDialog },
  props: {},
  data: () => ({
    page: 1,
    itemsPerPage: "6",
    searchExpression: "",
    isNoteDialogOpen: false,
    companyName: null,
    noteDescription: "",
    selectedDate: null,
    selectedTime: null,
    remindMe: false,
    currentItem: {},
    currentNoteId: null,
    isSaveNoteLoading: false,
    isOnEditMode: false,
    isDeleteDialogOpen: false
  }),
  created() {},
  mounted() {},
  methods: {
    ...mapActions(useGeneralStore, [
      "updateUserNotesListInStore",
      "removeNoteFromStore",
      "setToastMessage"
    ]),
    async saveNote() {
      this.isSaveNoteLoading = true;
      const noteData = {
        companyName: this.companyName,
        noteDescription: this.noteDescription,
        selectedDate: this.selectedDate,
        selectedTime: this.selectedTime,
        remindMe: this.remindMe
      };

      const { valid } = await this.$refs.newNoteForm.validate();
      if (valid) {
        if (this.isOnEditMode) {
          noteData.NoteId = this.currentNoteId;
          await this.updateCurrentNote(noteData);
        } else {
          await this.createNote(noteData);
        }
      }
    },
    onClearForm() {
      this.$refs.newNoteForm.reset();
      // reset
      this.isOnEditMode = false;
      this.currentNoteId = null;
    },
    async onEditNoteClick(note) {
      this.isNoteDialogOpen = true;
      // Display the selected note
      this.isOnEditMode = true;
      this.companyName = note.CompanyName;
      this.noteDescription = note.NoteDescription;
      this.selectedDate = note.SelectedDate;
      this.selectedTime = note.SelectedTime;
      this.remindMe = note.RemindMe;
      this.currentNoteId = note.NoteId;
    },
    async createNote(noteData) {
      const response = await createNewNote(noteData);
      if (response.success) {
        this.setToastMessage({
          type: "success",
          message: ToastMessages.SuccessMessages.Created
        });
        this.isSaveNoteLoading = false;
        this.updateUserNotesListInStore(response.data);
        this.onClearForm();
      }
    },
    async updateCurrentNote(noteData) {
      const response = await updateNote(noteData);
      if (response.success) {
        console.log("update res:", response.data);

        this.setToastMessage({
          type: "success",
          message: ToastMessages.SuccessMessages.Updated
        });
        this.isSaveNoteLoading = false;
        this.updateUserNotesListInStore(response.data);
        this.onClearForm();
      }
    },
    onDeleteIconClick(item) {
      this.currentItem = item;
      this.isDeleteDialogOpen = true;
    },
    async deleteItem(item) {
      const response = await removeNote(item);
      if (response.success) {
        this.removeNoteFromStore(item);
      }
      this.setToastMessage({
        type: "success",
        message: ToastMessages.SuccessMessages.Removed
      });
      // Reset current item and close dialog
      this.currentItem = {};
      this.isDeleteDialogOpen = false;
    },
    onWatchIconClick(item) {
      console.log("watch this:", item);
      this.$emit("onWatchNote", item);
    }
  },
  computed: {
    ...mapState(useGeneralStore, ["companiesList", "userNotesList"]),
    tableHeaders() {
      return [
        {
          title: "Id",
          key: "NoteId",
          width: "5%"
        },
        {
          title: "Attached To",
          key: "CompanyName"
          // width: "20%"
        },
        {
          title: "Created At",
          align: "center",
          key: "SelectedDate",
          width: "16%"
        },
        {
          title: "Reminder",
          align: "center",
          sortable: false,
          width: "5%"
        },
        // {
        //   title: "Watch",
        //   align: "center",
        //   sortable: false,
        //   width: "5%"
        // },
        {
          title: "Edit",
          align: "center",
          sortable: false,
          width: "5%"
        },
        {
          title: "Delete",
          align: "center",
          sortable: false,
          width: "5%"
        }
      ];
    },
    tableItems() {
      return this.userNotesList;
    },
    pageCount() {
      return Math.ceil(this.tableItems.length / this.itemsPerPage);
    },
    formRules() {
      return {
        required: (value) => !!value || "Field is required"
      };
    }
  },
  watch: {}
};
</script>

<style scoped lang="scss">
// .notes-table:deep(.v-table__wrapper) {
//   th:nth-child(3n) {
//     width: 24%;
//   }
// }
</style>
