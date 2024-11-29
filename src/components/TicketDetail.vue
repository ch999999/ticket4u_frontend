<template>

    <v-card :variant="variant" width="350" min-height="520" rounded="xl">
        <div class="ticket" id = "ticket">
              <div>
                <v-row>
                    <v-col cols="12">
                        <v-card-subtitle class="small-vertical-gap text-left">Date</v-card-subtitle>
                        <v-card-subtitle class="text-h5 white-text text-left" opacity="1">{{ date }}</v-card-subtitle>
                    </v-col>
                </v-row>
              </div>
              <div>
                <v-card-subtitle class="text-left">Time</v-card-subtitle>
                <v-card-subtitle class="text-h5 white-text text-left" opacity="1">{{ time }}</v-card-subtitle>
              </div>
              <div class="">
                  <v-card-subtitle class="small-vertical-gap text-left">Location</v-card-subtitle>
                  <v-card-subtitle class="text-h5 white-text text-left" opacity="1">{{ location }}</v-card-subtitle>
              </div>
              <div class="small-vertical-gap">
                  <v-card-subtitle class="text-left">Movie Title</v-card-subtitle>
                  <v-card-subtitle class="text-h5 text-column mb-5 text-left" opacity="1"><p class="text-content">{{ title }}</p></v-card-subtitle>
              </div>
              <div class="small-vertical-gap">
                  <v-card-subtitle class="text-left">Duration</v-card-subtitle>
                  <v-card-subtitle class="text-h5 text-column mb-5 text-left" opacity="1"><p class="text-content">{{ duration }} Minutes</p></v-card-subtitle>
              </div>
              <div class="small-vertical-gap">
                  <v-row>
                      <v-col cols="12">
                          <v-card-subtitle class="text-left">Seats ({{ seatCount }})</v-card-subtitle>
                          <v-card-subtitle class="text-h5 white-text text-left" opacity="1">{{ seatString }}</v-card-subtitle>
                      </v-col>
                  </v-row>
              </div>
              <div class="horizontal-centering small-vertical-gap">
                  <div class="vertical-align">
                      <v-btn class="text-none" color="#821DE7" width="220" @click="downloadTicket">Download Ticket</v-btn>
                  </div>
              </div>
        </div>
    </v-card>

</template>

  
<script setup>
import { defineProps, computed, ref, onMounted } from 'vue';
import jsPDF from 'jspdf';
import QRCode from 'qrcode';
const variant = ref("outlined")
const props = defineProps({
    date: {
        type: String,
        default: "Mon, 23 Oct 2023"
    },
    title: {
        type: String,
        default: "Movie title"
    },
    seats: {
        type: Array,
        default: () => []
    },
    time: {
        type: String,
        default: "14:40"
    },
    location: {
        type: String,
        default: "Cinema 1, Hall C"
    },
    duration:{
        type: Number,
        default: 50
    }
});
const seatString = computed(() => {
    const sorted = props.seats.sort((a, b) => a[0].localeCompare(b[0]) || parseInt(a.slice(1)) - parseInt(b.slice(1)));
    return sorted.join(', ')
});
const seatCount = computed(() => props.seats.length);

const generateRandomString = (length) => {
    const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    let result = '';
    for (let i = 0; i < length; i++) {
        result += characters.charAt(Math.floor(Math.random() * characters.length));
    }
    return result;
};

const generateNewQRCode = async () => {
    const qrCodeData = `Ticket for ${props.title} at ${props.location} on ${props.date} at ${props.time}`;
    return await QRCode.toDataURL(qrCodeData);
};

const downloadTicket = async () => {
    const doc = new jsPDF();
    doc.setFillColor(130, 29, 231); // Set the fill color to purple
    doc.rect(20, 0, 170, 15, 'F'); // Draw a filled rectangle
    doc.setTextColor(255, 255, 255); // Set the text color to white
    doc.setFont("helvetica", "bold");
    doc.text("tiket4u.", 105, 10, { align: 'center' }); // Center the text inside the rectangle
    
    doc.setTextColor(0, 0, 0); // Set the text color to black
    doc.setFont("helvetica", "normal");
    doc.text(`Movie Title: ${props.title}`, 20, 30);
    doc.text(`Location: ${props.location}`, 20, 40);
    doc.text(`Date: ${props.date}`, 20, 50);
    doc.text(`Time: ${props.time}`, 20, 60);
    doc.text(`Seats: ${seatString.value}`, 20, 70);

    const qrCodeUrl = await generateNewQRCode(); // Generate new QR code
    doc.addImage(qrCodeUrl, 'PNG', 80, 80, 50, 50); // Adjust the position and size as needed
  
    doc.save('ticket.pdf');
};

onMounted(() => {
    variant.value = "outlined"
    generateNewQRCode();
});
</script>

<style>
    @import '@/styles/isaac-styles.css';

    [v-cloak] {
  display: none;
}
</style>
