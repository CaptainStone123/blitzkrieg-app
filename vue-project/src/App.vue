<script>

import axios from 'axios';

export default {
  name : 'Chatbox',
  
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
      isChatboxHidden: true,  
      showChatMessage: false,
      profileImageSrc: '',
    };
  },
  mounted() {
        axios.get('https://blitzkrieg-node-server.vercel.app/api/getNameFromMongoDB')
        .then(response => {
          this.Name = response.data.Name;
        })
        .catch(error => {
          console.error('Error fetching Name:', error);
          console.log('Response status:', error.response ? error.response.status : 'N/A');
          console.log('Response data:', error.response ? error.response.data : 'N/A');
        });

        // chat message
        setTimeout(this.showChatFor5Seconds, 8000);    4
      
      
      
        axios.get('https://blitzkrieg-node-server.vercel.app/api/getImageFromMongoDB')
            .then(response => {
                const imageName = response.data.Image || 'stacy.png';
                const imageSrc = `/${imageName}`;

                console.log(imageSrc);
                
                this.loadImage(imageSrc);
            })
            .catch(error => {
                console.error('Error fetching image:', error);
            });
      },

 
  methods: {
    loadImage(src) {
      this.profileImageSrc = src;
    },
    deleteAll() {
        this.saveToLocalStorage();
        this.saveConversationToServer();//added NIER 101
        this.conversation = []; // Clear the conversation
      },
      saveToLocalStorage() { 
        window.localStorage.setItem('conversation', JSON.stringify(this.conversation));
      },
      loadFromLocalStorage() {
        // Load the conversation from local storage
        const storedconversation = window.localStorage.getItem('conversation');
        if (storedconversation) {
          this.conversation = JSON.parse(storedconversation);
        }
      },
      saveConversationToServer() {
        const userMessages = this.conversation.filter(message => message.role === 'user');

        if (userMessages.length === 0) {
          console.error('No user messages to save.');
          return;
        }
        
        axios.post('https://blitzkrieg-node-server.vercel.app/api/saveConversation', {
          id: null,
          duration: this.conversation.filter(item => item.role === 'user').length, 
          details: userMessages, 
        }, {
          headers: {
            "Content-Type": "application/json"
          }
        })
        .then(response => {
          console.log('Conversation data saved on the server:', response.data);
        })
        .catch(error => {
          console.error('Error saving conversation:', error);
        });
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
      // this.loadFromLocalStorage();
      window.addEventListener('beforeunload', this.saveConversationToServer);
    }
};
</script>

<template>
<div>
  <section class="chat-box" :style="{ display: isChatboxHidden ? 'none' : 'block' }">
    <div class="relative h-[100%] w-[100%]">

      <div class="minichat-container">
        <ul class="minichat-header relative flex justify-between rounded-tl-[.5rem] rounded-tr-[.5rem] bg-[#003075] py-[.5rem] px-[.8rem]">
          <li class="flex items-center">
             <img class="stacy" :src="profileImageSrc" alt=""><span class="name"><b>{{Name}}</b></span>
          </li>
          <li class="flex items-center">
            <i class="fa-solid fa-repeat text-[.9rem] mx-[.8rem]"  @click="deleteAll"></i>
            <i class="fa-solid fa-xmark text-[1.1rem]" @click="toggleChatbox"></i>
          </li>
        </ul>

        <div class="messagecontainer text-black flex flex-col flex-grow justify-start p-[.4rem]">
            <div v-for="(message, index) in conversation" :key="index" class="message-container">
              <div :class="[message.role === 'user' ? 'flex user-message justify-end' : 'flex bot-message justify-start ']">
                 <span class="message text-[15px] mx-[10px]" v-html="message.content"></span>
              </div>
            </div>
            <div v-if="isLoading" class="loading-message text-black m-[10px]">
               <div class="loading-box">
                <span class="dot"></span>
                <span class="dot"></span>
                <span class="dot"></span>
              </div>
            </div>
        </div>
        <hr>
        <div class="minichatfooter flex ">
          <span class="bg-[#f3f3f5] w-[100%]">
            <input type="text" v-model="input" @keyup.enter="fetchData"  class="bg-transparent w-[85%] p-[.5rem] text-black text-[15px] focus:border-transparent  focus:outline-none focus:ring-0 placeholder-gray" placeholder="Input question here">
            <button type="submit" @click="fetchData" class="w-[15%]">
              <i class="fa-solid fa-paper-plane text-[gray]"></i>
            </button>
          </span>
         
        </div>
      </div>

    </div>
  </section>

  <div v-if="isChatboxHidden" @click="showChatbox" class="show-chat-btn">
    <section v-if="showChatMessage" class="chat-btn-text">
      <span>Hello I'm {{ Name }}! you can ask me if you have any questions</span> <!--NIER-101 changed "Stacy" to {{ Name }}-->
      <div class="chat-arrow"></div>
     </section>
    <section class="chat-btn-img">
      <img :src="profileImageSrc" alt="">
    </section>
   </div>
</div>
 
</template>

<style>
.chat-btn-text[style*="none"] {
  display: none; /* Hide the message when 'showChatMessage' is false */
}

.show-chat-btn {
  position: absolute;
  bottom: 25px; right: 25px;
  border: none;
  cursor: pointer;
  height: 75px;
  display: flex;  justify-content: center; align-items: center;
}
.chat-btn-text{
  height: 100%;
  display: flex;
  align-items: center;
  margin: .5rem;
}
.chat-btn-text span{
  background-color: white;
  border-radius: 5px;
  height: 100%;
  display: flex;
  justify-content: center; align-items: center;
  padding: 1rem;
}
.chat-arrow{
  width: 5px; height: 5px;
  border-left: 10px solid transparent; border-right: 10px solid transparent;  
  border-bottom: 20px solid white;  
  transform: rotate(90deg);
}
.chat-btn-img{
  height: 65px; width: 65px;
  background-color: white;
  padding:10px;
  border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
}
.chat-btn-img img{
 height: 55px;    
}

/* CHATBOX  */
.chat-box{
  position: absolute;
  bottom: 0; right: 8px;
  height: 27rem; width: 20rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  color: white;
  border-top-left-radius: 0.5rem;
  border-top-right-radius: 0.5rem;
}

.minichat-container{
  border-top-left-radius: 0.5rem;
  border-top-right-radius: 0.5rem;
  background-color: white;
  display: flex;
  flex-direction: column;
  height: 100%;
}

/* CHAT HEADER */

.stacy {
  width: 100px;
  height: auto;
  position: absolute;
  margin-top: -82px;
  margin-left: -10px;
}
.name {
  margin-left: 95px;
}
.message-container {
  margin-bottom: 1rem;
}
.user-message .message {
  background-color: #0084ff;
  padding: 0.5rem;
  border-radius: 0.5rem;
}
.bot-message .message {
  background-color: lightgray;
  padding: 0.5rem;
  border-radius: 0.5rem;
  color: black;
}
.minichat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.messagecontainer {
  flex-grow: 1;
  overflow-y: auto;
}

.minichatfooter {
  padding: 0.5rem;   
}
 
/* LOADING BOX  */
.loading-box {
  width: 50px;height: 30px;
  display: flex; align-items: center; justify-content: center;
  background-color: #f0f0f0;
  overflow: hidden; 
  border: 1px solid #ccc;border-radius: 25px; border: none;
}

.dot-container {
  display: flex; align-items: center; justify-content: center;
}

.dot {
  width: 5px; height: 5px;
  background-color: #8c9095;
  border-radius: 50%;
  margin: 0 3px;
  animation: jump 1s infinite alternate;
}
.dot:nth-child(2) {animation-delay: 0.2s;}
.dot:nth-child(3) {animation-delay: 0.4s;}
@keyframes jump {0%, 100% {transform: translateY(0);}50% {transform: translateY(-5px);}}


</style>
