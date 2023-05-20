<template>
  <div>
    <NewMeetingForm @added="addNewMeeting($event)"></NewMeetingForm>
    <span v-if="meetings.length === 0">Brak zaplanowanych spotkań.</span>
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
  props: {username: String, meetings: Array},
  data() {
    return {

    };
  },
  methods: {
    addNewMeeting(meeting) {
      axios.post('/api/meetings', meeting)
          .then(response => {
            this.message = ("Udało się dodać spotkanie")
            this.meetings.push({
              id: response.data.id,
              ...meeting
            })
          })
          .catch(response => {
            this.message = ("Nie udało się dodać spotkania")
          });
    },
    addMeetingParticipant(meeting) {
      axios.post("/api/meetings/" + meeting.id + "/participants", {login: this.username})
          .then(response => {
            this.message = ("Udało się dodać uczestnika spotkania")
            meeting.participants.push({login: this.username});
          })
          .catch(response => {
            this.message = ("Nie udało się dodać uczestnika spotkania")
          });
    },
    removeMeetingParticipant(meeting) {
      axios.delete("/api/meetings/" + meeting.id + "/participants/" + this.username)
          .then(response => {
            this.message = ("Udało się usunąć uczestnika spotkania")
            meeting.participants.splice(meeting.participants.findIndex(p => p.login === this.username), 1);
          })
          .catch(response => {
            this.message = ("Nie udało się usunąć uczestnika spotkania")
          });
    },
    deleteMeeting(meeting) {
      axios.delete('/api/meetings/' + meeting.id)
          .then(response => {
            this.message = ("Udało się usunąć spotkanie")
            this.meetings.splice(this.meetings.indexOf(meeting), 1);
          })
          .catch(response => {
            this.message = ("Nie udało się dodać spotkania")
          });
    },
  }
}
</script>
