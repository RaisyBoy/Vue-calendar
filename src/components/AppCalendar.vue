<template>
  <div>
    <div id="calendar" class="height"></div>
    
    <!-- Modal for creating event -->
    <div v-if="isModalOpen" class="modal" :style="modalStyle">
      <button class="close-button" @click="closeModal">×</button>
      <div class="modal-content">
        <input v-model="eventTitle" placeholder="Enter title" class="input" />
        <input v-model="eventDate" class="input" />
        <div class="button-container">
          <input v-model="startTime" type="time" class="input-time" />
          <span>-</span>
          <input v-model="endTime" type="time" class="input-time" />
        </div>
        <input v-model="eventDescription" placeholder="Enter description" class="input" />
        <div class="button-container">
          <button @click="closeModal" class="button-cancel">Cancel</button>
          <button @click="saveEvent" class="button-add">Save</button>
        </div>
      </div>
    </div>

    <!-- Modal for editing event -->
    <div v-if="isEditOpen" class="modal" :style="modalStyle">
      <button class="close-button" @click="closeEditEvent">×</button>
      <div class="modal-content">
        <input v-model="editTitle" placeholder="Enter title" class="input" :disabled="!isAllowEdit" />
        <input v-model="editDate" class="input" :disabled="!isAllowEdit" />
        <div class="button-container">
          <input v-model="editStartTime" type="time" class="input-time" :disabled="!isAllowEdit" />
          <span>-</span>
          <input v-model="editEndTime" type="time" class="input-time" :disabled="!isAllowEdit" />
        </div>
        <input v-model="editDescription" placeholder="Enter description" class="input" :disabled="!isAllowEdit" />
        <div class="button-container">
          <button @click="deleteEditEvent" class="button-cancel">Discard</button>
          <button v-if="!isAllowEdit" @click="allowEdit" class="button-add">Edit</button>
          <button v-else @click="saveUpdate" class="button-add">Update</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { createCalendar, createViewMonthGrid, createViewDay, createViewWeek } from '@schedule-x/calendar';
import { createDragAndDropPlugin } from '@schedule-x/drag-and-drop';
import { createEventsServicePlugin } from '@schedule-x/events-service';
import '@schedule-x/theme-default/dist/index.css';

const plugins = [createDragAndDropPlugin(), createEventsServicePlugin()];

const isModalOpen = ref(false);
const isEditOpen = ref(false);
const isAllowEdit = ref(false);

// Event data for creating and editing events
const eventTitle = ref('');
const eventDescription = ref('');
const eventDate = ref('');
const startTime = ref('');
const endTime = ref('');

const editTitle = ref('');
const editDescription = ref('');
const editId = ref(null);
const editDate = ref('');
const editStartTime = ref('');
const editEndTime = ref('');

const modalStyle = ref({ top: '0px', left: '0px' });
const calendarRef = ref(null);

// Helper function to validate event fields
const isValidEvent = (title, description) => {
  return title.length <= 30 && description.length <= 30 && title && description;
};

// Allow editing in the edit modal
const allowEdit = () => {
  isAllowEdit.value = true;
};

// Save updated event
const saveUpdate = () => {
  if (!isValidEvent(editTitle.value, editDescription.value) || !editDate.value || !editStartTime.value || !editEndTime.value) {
    alert('Please fill in all fields and ensure title and description are not more than 30 characters.');
    return;
  }

  const calendar = calendarRef.value;
  calendar.eventsService.update({
    title: editTitle.value,
    start: `${editDate.value} ${editStartTime.value}`,
    end: `${editDate.value} ${editEndTime.value}`,
    description: editDescription.value,
    id: editId.value,
  });
  closeEditEvent();
};

// Open edit modal for an event
const openEdit = (event, date) => {
  if (isModalOpen.value || isEditOpen.value) return;

  const [startDate, startTimeValue] = date.start.split(' ');
  const [, endTimeValue] = date.end.split(' ');

  editTitle.value = date.title;
  editId.value = date.id;
  editDate.value = startDate;
  editStartTime.value = startTimeValue;
  editEndTime.value = endTimeValue;
  editDescription.value = date.description;

  const { clientX, clientY } = event;
  const clickedRect = event.target.getBoundingClientRect();
  const offsetX = clientX - clickedRect.left;
  const offsetY = clientY - clickedRect.top;

  modalStyle.value = {
    top: `${clickedRect.top + window.scrollY + offsetY + 30}px`,
    left: `${clickedRect.left + window.scrollX + offsetX - 150}px`,
  };

  isEditOpen.value = true;
};

// Open create modal
const openModal = (event, date) => {
  if (isModalOpen.value || isEditOpen.value) return;

  eventDate.value = date;
  const { clientX, clientY } = event;

  const clickedRect = event.target.getBoundingClientRect();
  const offsetX = clientX - clickedRect.left;
  const offsetY = clientY - clickedRect.top;

  modalStyle.value = {
    top: `${clickedRect.top + window.scrollY + offsetY + 30}px`,
    left: `${clickedRect.left + window.scrollX + offsetX - 150}px`,
  };

  isModalOpen.value = true;
};

// Delete event in edit mode
const deleteEditEvent = () => {
  const calendar = calendarRef.value;
  calendar.eventsService.remove(editId.value);
  closeEditEvent();
};

// Close edit modal
const closeEditEvent = () => {
  isEditOpen.value = false;
  editTitle.value = '';
  editId.value = null;
  editDate.value = '';
  editStartTime.value = '';
  editEndTime.value = '';
  isAllowEdit.value = false;
};

// Close create modal
const closeModal = () => {
  isModalOpen.value = false;
  eventTitle.value = '';
  eventDescription.value = '';
  startTime.value = '';
  endTime.value = '';
};

// Save event to calendar
const saveEvent = () => {
  if (!isValidEvent(eventTitle.value, eventDescription.value) || !eventDate.value || !startTime.value || !endTime.value) {
    alert('Please fill in all fields and ensure title and description are not more than 30 characters.');
    return;
  }

  const calendar = calendarRef.value;
  const uniqueId = Date.now();
  calendar.eventsService.add({
    title: eventTitle.value,
    start: `${eventDate.value} ${startTime.value}`,
    end: `${eventDate.value} ${endTime.value}`,
    description: eventDescription.value,
    id: uniqueId,
  });
  closeModal();
};

// Initialize calendar on mount
onMounted(() => {
  const config = {
    views: [createViewMonthGrid(), createViewDay(), createViewWeek()],
    events: [
      {
        id: 1,
        title: 'Coffee with John',
        start: '2025-04-06 10:05',
        end: '2025-04-06 10:35',
        color: 'red',
      },
    ],
    callbacks: {
      onClickDate: (date) => {
        eventDate.value = date;
        openModal(event, date);
      },
      onEventClick: (calendarEvent) => openEdit(event, calendarEvent),
      onClickDateTime: (dateTime) => {
        const dateOnly = dateTime.split(' ')[0];
        eventDate.value = dateOnly;
        openModal(event, dateOnly);
      },
    },
    plugins,
  };

  const calendar = createCalendar(config);
  calendarRef.value = calendar;

  const calendarElement = document.getElementById('calendar');
  if (calendarElement) {
    calendar.render(calendarElement);
  } else {
    console.error('Element with id "calendar" not found!');
  }
});
</script>

<style scoped>
.modal {
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 15px;
  width: 201px;
  height: 275px;
  box-shadow: 0 24px 38px 3px rgba(0, 0, 0, 0.14), 0 9px 46px 8px rgba(0, 0, 0, 0.12), 0 11px 15px -7px rgba(0, 0, 0, 0.2);
  border: 1px solid black;
}

.input {
  max-width: 197px;
  min-width: 197px;
  padding-top: 10px;
  padding-bottom: 10px;
  margin-top: 20px;
  border: none;
  border-bottom: 2px solid #ccc;
}

.input-time {
  max-width: 80px;
  min-width: 80x;
  padding-top: 10px;
  padding-bottom: 10px;
  margin-top: 10px;
  border: none;
  border-bottom: 2px solid #ccc;
}

.button-add {
  padding: 10px 15px;
  background-color: white;
  color: #6A6996;
  width: 70px;
  align-content: center;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.button-cancel {
  padding: 10px 15px;
  background-color: white;
  color: #FF5F5F;
  width: 70px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.button-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  margin-top: 10px;
}

.height {
  height: 80vh;
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  width: 28px;
  height: 28px;
  padding-left: 6px;
  background-color: white;
  border: none;
  border-radius: 50%;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  line-height: 1;
  color: #333;
  transition: background-color 0.2s ease;
}

button:hover {
  background-color: #e0e0e0;
}
</style>
