
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
 

  </header>
</template>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
