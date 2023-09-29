
<script>

import axios from 'axios';

export default {
   
  data() {
    return {
      Name: '',
      input: '',
      isLoading: false,
      error: null,
      message: null,
      messageUser: null,
      uiData: {},
      conversation:[],
      isChatboxHidden: true, // Initially, the chatbox is not hidden
      showChatMessage: false,
    };
  },
  mounted() {
        axios.get('http://localhost:3000/api/getNameFromMongoDB')
        .then(response => {
          this.Name = response.data.Name;
        })
        .catch(error => {
          console.error('Error fetching Name:', error);
          console.log('Response status:', error.response ? error.response.status : 'N/A');
          console.log('Response data:', error.response ? error.response.data : 'N/A');
        });

        // chat message
        setTimeout(this.showChatFor5Seconds, 8000);    },
 
  methods: {
    deleteAll() {
        this.conversation = []; // Clear the conversation
        this.saveToLocalStorage();
      },
      saveToLocalStorage() {
        // Save the conversation to local storage
        window.localStorage.setItem('conversation', JSON.stringify(this.conversation));
      },
      loadFromLocalStorage() {
        // Load the conversation from local storage
        const storedconversation = window.localStorage.getItem('conversation');
        if (storedconversation) {
          this.conversation = JSON.parse(storedconversation);
        }
      },
     async fetchData() {
        this.isLoading = true;
        const options = {
          method: "POST",
          body: JSON.stringify({
            messages: this.input
          }), 
          headers: {
            "Content-Type": "application/json"
          }
        };
        try {
          const response = await fetch('http://localhost:5000/completions', options);
          const data = await response.json();
          const conMessage = data.choices[0].message;
  
          console.log("Testing")
  
          this.messageUser = { role: "user", content: this.input};
          this.message = { role: conMessage.role, content: conMessage.content};
          
          this.conversation.push(this.messageUser, this.message);
          this.userInput = '';  
          this.saveToLocalStorage();
   
          console.log(this.conversation);
         } catch (error) {
          this.error = 'An error occurred while fetching data.';
        } finally {
          this.isLoading = false;
        }
      },
 
    toggleChatbox() {
      // Toggle the chatbox visibility
      this.isChatboxHidden = !this.isChatboxHidden;
    },
    showChatbox() {
      // Show the chatbox and hide the "click me to show" button
      this.isChatboxHidden = false;
    },
    // chat message
    showChatFor5Seconds() {
      this.showChatMessage = true;
      setTimeout(() => {
        this.showChatMessage = false;
      }, 5000);
    },
    
  },
  created() {
      // Load the conversation from local storage when the component is created
      this.loadFromLocalStorage();
    }
};
</script>
<template>
  <header>
 dsdsd

  </header>
</template>

<style scoped>

</style>
