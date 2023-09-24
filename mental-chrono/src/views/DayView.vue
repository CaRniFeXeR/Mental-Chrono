<template>
    <div class="calendar-day-view">
      <svg class="clock" viewBox="0 0 300 300" @click="onClockClick($event)">
        <!-- Clock background -->
        <circle cx="100" cy="100" r="95" fill="none" stroke="#e0e0e0" stroke-width="2" />
        
        <!-- Appointments -->
        <path
          v-for="appointment in appointments"
          :d="getPathData(appointment)"
          class="appointment"
          :data-id="appointment.id"
          :key="appointment.id"
          @mouseover="onAppointmentHover(appointment)"
          @mouseout="onAppointmentLeave"
        />
        
        <!-- Center details -->
   <text v-if="selectedAppointment" x="100" y="60" text-anchor="middle" class="title">{{ selectedAppointment.name }}</text>
    <text v-if="selectedAppointment" x="100" y="80" text-anchor="middle" class="duration">{{ `${decimalHourToTimeString(selectedAppointment.start)} - ${decimalHourToTimeString(selectedAppointment.end)}` }}</text>
    <text v-if="selectedAppointment" x="100" y="100" text-anchor="middle" class="location">{{ selectedAppointment.location }}</text>
    <text v-if="selectedAppointment" x="100" y="120" text-anchor="middle" class="description">{{ selectedAppointment.description }}</text>
  
         <!-- Hourly ticks -->
    <line
      v-for="hour in 12"
      :key="hour"
      :x1="100 + 90 * Math.cos(Math.PI * (hour * 30 - 90) / 180)"
      :y1="100 + 90 * Math.sin(Math.PI * (hour * 30 - 90) / 180)"
      :x2="100 + 95 * Math.cos(Math.PI * (hour * 30 - 90) / 180)"
      :y2="100 + 95 * Math.sin(Math.PI * (hour * 30 - 90) / 180)"
      stroke="#e0e0e0"
      class="hour-tick"
      :alt="hour"
      stroke-width="1"
    />
  
      <line
      :x1="100 + 90 * Math.cos(Math.PI * (currentTimeAngle - 90) / 180)"
      :y1="100 + 90 * Math.sin(Math.PI * (currentTimeAngle - 90) / 180)"
      :x2="100 + 95 * Math.cos(Math.PI * (currentTimeAngle - 90) / 180)"
      :y2="100 + 95 * Math.sin(Math.PI * (currentTimeAngle - 90) / 180)"
      stroke="#FF5722"
      stroke-width="2"
    />
  
    <!-- Start time label for hovered or selected appointment -->
    <text 
      v-if="hoveredAppointment || selectedAppointment"
      :x="105 + 110 * Math.cos(Math.PI * (getStartAngle(hoveredAppointment || selectedAppointment) - 90) / 180)"
      :y="105 + 110 * Math.sin(Math.PI * (getStartAngle(hoveredAppointment || selectedAppointment) - 90) / 180)"
      text-anchor="middle"
      alignment-baseline="middle"
      class="appointment-time"
    >
      {{ decimalHourToTimeString(hoveredAppointment ? hoveredAppointment.start : selectedAppointment.start) }}
    </text>
  
    <!-- End time label for hovered or selected appointment -->
    <text 
      v-if="hoveredAppointment || selectedAppointment"
      :x="105 + 110 * Math.cos(Math.PI * (getEndAngle(hoveredAppointment || selectedAppointment) - 90) / 180)"
      :y="105 + 110 * Math.sin(Math.PI * (getEndAngle(hoveredAppointment || selectedAppointment) - 90) / 180)"
    
      text-anchor="middle"
      alignment-baseline="middle"
      class="appointment-time"
    >
      {{ decimalHourToTimeString(hoveredAppointment ? hoveredAppointment.end : selectedAppointment.end) }}
    </text>
    <text v-if="hoveredAppointment" x="100" y="140" text-anchor="middle" class="hover-title">{{ hoveredAppointment.name }}</text>
      </svg>
  
    
      
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        appointments: [
          {
            id: 1,
            start: 13,
            end: 15,
            name: "Lunch with Cousin Stark",
            description: "They probably not able to talk in english",
            location: "Hot pot restaurant"
          },
           {
            id: 2,
            start: 18,
            end: 19.5,
            name: "Chess with dad",
            description: "",
            location: "Tablet"
          },
          // ... more appointments
        ],
        selectedAppointment: null,
        hoveredAppointment: null
      };
    },
    computed: {
      currentTimeAngle() {
        const now = new Date();
        const hour = now.getHours();
        const minute = now.getMinutes();
        return (hour + minute / 60) * 30;
      }
    },
    methods: {
    decimalHourToTimeString(decimalHour) {
      // Extract the whole number (hour part)
      let hour = Math.floor(decimalHour);
      
      // Get the decimal part and multiply by 60 to get minutes
      let minute = Math.round((decimalHour - hour) * 60);
      
      // Format the values into a string
      return `${hour}:${minute.toString().padStart(2, '0')}`;
  },
      onClockClick(event) {
        if (event.target && event.target.classList.contains('appointment')) {
          const appointmentId = parseInt(event.target.getAttribute('data-id'), 10);
          this.selectedAppointment = this.appointments.find(app => app.id === appointmentId);
        } else {
          this.selectedAppointment = null; // Deselect if clicked outside an appointment
        }
      },
      getPathData(appointment) {
        const startAngle = (appointment.start) * 30; // -8 because the calendar starts from 8am
        const endAngle = (appointment.end) * 30;
  
        const x1 = 100 + 95 * Math.cos(Math.PI * (startAngle - 90) / 180);
        const y1 = 100 + 95 * Math.sin(Math.PI * (startAngle - 90) / 180);
  
        const x2 = 100 + 95 * Math.cos(Math.PI * (endAngle - 90) / 180);
        const y2 = 100 + 95 * Math.sin(Math.PI * (endAngle - 90) / 180);
  
        const largeArcFlag = endAngle - startAngle <= 180 ? 0 : 1;
  
        return `M ${x1} ${y1} A 95 95 0 ${largeArcFlag} 1 ${x2} ${y2}`;
      },
        onAppointmentHover(appointment) {
        this.hoveredAppointment = appointment;
      },
      onAppointmentLeave() {
        this.hoveredAppointment = null;
      },
       getStartAngle(appointment) {
        return (appointment.start) * 30;
      },
      getEndAngle(appointment) {
        return (appointment.end) * 30;
      }
    }
  };
  </script>
  
  <style scoped>
  .calendar-day-view {
    width: 300px;
    height: 300px;
    position: relative;
  }
  
  .clock {
    width: 100%;
    height: 100%;
  }
  
  .appointment {
    fill: none;
    stroke: #2196F3;
    stroke-width: 5;
    cursor: pointer;
    transition: stroke 0.3s;
    stroke-linecap: round;
    transition: 0.25s ease;
  }
  
  .appointment:hover {
    stroke: #FF5722;
     stroke-width: 10;
  }
  
  .clock text {
    font-family: 'Arial', sans-serif;
    fill: #333;
  }
  
  .title {
    font-size: 12px;
    font-weight: bold;
  }
  
  .duration {
  font-size: 11px;
  }
  
  .location {
    font-size: 10px;
    fill: #FF5722;
  }
  
  .description {
    font-size: 8px;
    font-style: italic;
  }
  
  .hour-tick:hover{
  stroke-width : 4px
  }
  
  .hover-title {
    font-size: 12px;
    font-weight: bold;
    fill: #FF5722;
  }
  
  .appointment-time {
  font-size : 10px; 
  }
  </style>
  