<template>
  <div>
    <NewMeetingForm @added="addNewMeeting($event)"></NewMeetingForm>
    <div class="error" v-if="idError">Spotkanie o podanej nazwie istnieje. Wprowadź inną nazwę</div>
    <span v-if="meetings.length == 0">Brak zaplanowanych spotkań.</span>
    <h3 v-else>Zaplanowane zajęcia ({{ meetings.length }})</h3>
    <MeetingsList :meetings="meetings"
                  :username="username"
                  @attend="addMeetingParticipant($event)"
                  @unattend="removeMeetingParticipant($event)"
                  @delete="deleteMeeting($event)"></MeetingsList>
  </div>
</template>

<script>
import NewMeetingForm from "./NewMeetingForm";
import MeetingsList from "./MeetingsList";
import axios from "axios";

export default {
  components: {NewMeetingForm, MeetingsList},
  props: {
    username: String
  },
  data() {
    return {
      meetings: [],
      idError: false
    }
  },
  created: async function() {
  let importedDbDataMeetings = [];
  let importedDbParticipants = [];
  await axios.get('/api/meetings')
        .then(response => {
          importedDbDataMeetings = response.data;
            console.log(importedDbDataMeetings);
            importedDbDataMeetings.forEach(element => {
                axios.get('/api/meetings/' + element.id + '/participants')
                    .then(response => {
                          importedDbParticipants = response.data;
                          let participants = [];
                          importedDbParticipants.forEach(element => {
                              participants.push(element.login);
                          });
                          element.participants = participants;

                          this.meetings.push(element);
                    })
                    .catch(() => console.log('Uczestnicy spotkania [' + element.title + '] nie zostali pobrani z bazy danych'));
                });
        })
        .catch(() => console.log('Spotkania nie zostały pobrane z bazy danych'));
  },
  methods: {
    addNewMeeting(meeting) {
      if(meeting.description == null) {
        meeting.description = '';
      }

      axios.post('/api/meetings', meeting)
          .then(response => {
              meeting.id = response.data.id;
              console.log('Dodano spotkanie [' + meeting.title + ']');
              this.meetings.push(meeting);
              this.idError = false;
          })
          .catch(() => {
              this.idError = true;
              console.log('Spotkanie o nazwie [' + meeting.title + '] już istnieje');
          });
      console.log(meeting);
    },
    addMeetingParticipant(meeting) {
      const meetingIndex = this.meetings.indexOf(meeting);
      const participantsList = this.meetings.at(meetingIndex).participants;
      
      axios.post('/api/meetings/' + meeting.id + '/participants', {login:this.username})
          .then(response => console.log('Dodano uczestnika [' + this.username + '] do spotkania [' + meeting.title + ']'))
          .catch(() => 'Uczestnik nie został dodany do spotkania [' + meeting.title + ']');
      participantsList.push(this.username);
      console.log(this.meetings);
    },
    removeMeetingParticipant(meeting) {
      axios.delete('/api/meetings/' + meeting.id + '/participants/' + this.username)
          .then(response => console.log('Usunięto uczestnika [' + this.username + '] ze spotkania [' + meeting.title + ']'))
          .catch(() => 'Uczestnik nie został usunięty ze spotkania [' + meeting.title +']');
      meeting.participants.splice(meeting.participants.indexOf(this.username), 1);
    },
    deleteMeeting(meeting) {
      axios.delete('/api/meetings/' + meeting.id)
          .then(response => console.log('Usunięto spotkanie [' + meeting.title + ']'))
          .catch(() => 'Spotkanie [' + meeting.title + '] nie zostało usunięte');
      this.meetings.splice(this.meetings.indexOf(meeting), 1);
      this.error = false;
    }
  }
}
</script>

<style scoped>
.error {
  padding-top: -40px;
  padding-bottom: 20px;
  color: #F00;
}
</style>
