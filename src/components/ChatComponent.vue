<template>
  <div class="chat-container">
    <div class="chat_container_header">
      <p>LinkedIn Support</p>
    </div>
    <div class="conversation_wrapper">
      <ul id="example-1">
        <li v-for="message in messages" v-bind:key="message.id">
          <div class="message__container">
            <div class="message_author" v-if="message.type === 'outgoing'">You</div>
            <div class="message_author" v-else-if="message.type === 'incoming'">Support</div>
            <div clas="message__recieved">
              <span>{{ message.text }}</span>
            </div>
          </div>
        </li>
      </ul>
    </div>
    <div class="chat_actions_wrapper">
      <div class="text_input_wrapper">
        <textarea
          id="chat_text"
          name="messageSend"
          placeholder="Send message"
          v-model="outgoingMessage"
        />
      </div>
      <div class="buttons_wrapper">
        <button id="send_chat_button" type="submit" @click="sendMessage">Send</button>
      </div>
    </div>
    <button @click="startChat">Start Chat</button>
    <button @click="getIncomingMessages">Get Messages</button>
  </div>
</template>

<script>
import axios from "axios";
// const questionType = {
//   NAME: 'name',
//   EMAIL: 'email',
//   SUBJECT: 'subject'
// };

const status = {
  SLEEPING: "sleeping",
  SCREENING: "screening",
  CHATTING: "chatting"
};

export default {
  name: "ChatComponent",
  data: function() {
    return {
      messages: [],
      nextIndex: 0,
      chat: {
        chatUrl: "",
        chatAccessToken: "",
        refreshToken: "",
        sessionId: "",
        contactId: "",
        status: "",
        statusText: ""
      },
      pointOfContact: "bc75e6a9-d017-47ce-ba70-00e4e14cc467",
      outgoingMessage: "",
      status: status.SLEEPING,
      currentScreeningQuestion: -1,
      initMessage:
        "Thank you for contacting support, before I transfer you, there are a few things I need to ask you.",
      screeningQuestions: [
        // {
        //   id: 1,
        //   type: questionType.NAME,
        //   question: 'Can you please tell me your first and last name?',
        //   answer: ''
        // },
        // {
        //   id: 2,
        //   type: questionType.EMAIL,
        //   question: 'Can you please tell me your email address?',
        //   answer: ''
        // },
        // {
        //   id: 3,
        //   type: questionType.SUBJECT,
        //   question: 'Is this regarding billing?',
        //   answer: ''
        // },
        // {
        //   id: 4,
        //   type: questionType.SUBJECT,
        //   question:
        //     'And finally, can you please tell me what I may assist you with?',
        //   answer: ''
        // }
      ]
    };
  },
  props: {
    firstName: String,
    lastName: String,
    emailAddress: String,
    businessLine: String
  },
  computed: {},
  methods: {
    sendMessage() {
      switch (this.status) {
        case status.SCREENING:
          if (
            this.screeningQuestions[this.currentScreeningQuestion] &&
            this.screeningQuestions[this.currentScreeningQuestion].answer === ""
          ) {
            this.screeningQuestions[
              this.currentScreeningQuestion
            ].answer = this.outgoingMessage;

            this.messages.push({
              type: "outgoing",
              text: this.outgoingMessage
            });
            this.currentScreeningQuestion++;
            console.log(this.screeningQuestions.length);

            console.log(this.currentScreeningQuestion);
            if (
              this.currentScreeningQuestion === this.screeningQuestions.length
            ) {
              this.status = status.CHATTING;
            } else {
              setTimeout(() => {
                this.messages.push({
                  type: "incoming",
                  text: this.screeningQuestions[this.currentScreeningQuestion]
                    .question
                });
              }, 2000);
            }
          }
          break;
        case status.CHATTING: {
          const headers = {
            //Use access_token previously retrieved from inContact token service
            Authorization: "Bearer " + this.chat.chatAccessToken,
            "content-Type": "application/json"
          };

          const requestBody = {
            label: "test label",
            message: this.outgoingMessage
          };

          const sendTextChatUrl = `${this.chat.chatUrl}/${this.chat.sessionId}/send-text`;
          const newMessage = axios.post(sendTextChatUrl, requestBody, {
            headers: headers
          });

          newMessage.then(
            response => {
              if (response.status === 202) {
                console.log("message:", this.outgoingMessage);
                this.messages.push({
                  type: "outgoing",
                  text: this.outgoingMessage
                });
              }
              console.log(response);
              this.outgoingMessage = "";
            },
            error => {
              console.log(error);
            }
          );
          break;
        }
        default:
          break;
      }
    },
    getIncomingMessages() {
      const headers = {
        //Use access_token previously retrieved from inContact token service
        Authorization: "Bearer " + this.chat.chatAccessToken,
        "content-Type": "application/x-www-form-urlencoded"
      };

      const getMessagesUrl = `${this.chat.chatUrl}/${this.chat.sessionId}?timeout=30`;
      const newMessage = axios.get(getMessagesUrl, {
        headers: headers
      });

      newMessage.then(
        response => {
          if (response.status === 200) {
            const messages = response.data.messages;
            console.log(messages[this.nextIndex]);
            if (
              messages[this.nextIndex].Type &&
              messages[this.nextIndex].Type === "Status"
            ) {
              this.messages.push({
                type: "incoming",
                text: "Waiting"
              });

              this.nextIndex++;
            } else {
              if (
                messages[this.nextIndex].PartyTypeId === 1 ||
                messages[this.nextIndex].PartyTypeId === 2
              ) {
                this.messages.push({
                  type: "incoming",
                  text: messages[this.nextIndex].Text
                });
                this.nextIndex++;
              } else if (messages[this.nextIndex].PartyTypeId === 3) {
                this.nextIndex++;
              }
            }
          }
        },
        error => {
          console.log(error);
        }
      );
    },
    startChat() {
      this.getChatToken().then(
        response => {
          const data = response.data;
          this.chat.chatUrl = `${data.resource_server_base_uri}services/v17.0/contacts/chats`;
          this.chat.chatAccessToken = data.access_token;
          this.chat.refreshToken = data.refresh_token;
          const requestBody = {
            pointOfContact: this.pointOfContact,
            fromAddress: "test@example.com",
            parameters: ["", this.firstName, this.lastName, "Billing"],
            mediaType: 3
          };
          const headers = {
            //Use access_token previously retrieved from inContact token service
            Authorization: "Bearer " + this.chat.chatAccessToken,
            "content-Type": "application/json"
          };

          const newChat = axios.post(this.chat.chatUrl, requestBody, {
            headers: headers
          });

          newChat.then(
            response => {
              const data = response.data;
              if (response.status === 202) {
                this.chat.sessionId = data.chatSessionId;
                this.chat.contactId = data.contactId;
                this.status = status.CHATTING;

                this.getIncomingMessages();

                setInterval(
                  function() {
                    this.getIncomingMessages();
                  }.bind(this),
                  5000
                );
              }
              console.log(response);
            },
            error => {
              console.log(error);
            }
          );
        },
        error => {
          console.log(error);
        }
      );
    },
    getChatToken() {
      const URL =
        "https://api-b2.incontact.com/InContactAuthorizationServer/Token";

      const headers = {
        Accept: "application/json",
        Authorization: "someAuthorization",
        "Content-Type": "application/json"
      };

      const requestBody = {
        grant_type: "password",
        username: "someUserName",
        password: "somePassword",
        scope: ""
      };

      return axios.post(URL, requestBody, {
        headers: headers
      });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  margin: 0 10px;
}
a {
  color: #42b983;
}
.chat-container {
  width: 370px;
  height: calc(100% - 120px);
  max-height: 590px;
  position: fixed;
  right: 25px;
  bottom: 100px;
  box-sizing: border-box;
  box-shadow: 0px 7px 40px 2px rgba(148, 149, 150, 0.1);
  background: white;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  transition: 0.3s ease-in-out;
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  box-shadow: 0 2px 8px 2px rgba(0, 0, 0, 0.3);
  border-radius: 2px;
}
.chat_container_header {
  float: left;
  background-color: #283e4a;
  width: 100%;
  height: 40px;
  text-align: left;
}
.chat_container_header > p {
  font-size: 16px;
  color: white;
  margin: 10px;
}
.conversation_wrapper {
  background-color: white;
  opacity: 50%;
  padding: 1px;
  height: 80%;
  overflow: scroll;
}
.chat_actions_wrapper {
  height: 20%;
}
.text_input_wrapper {
  height: 58px;
}
.buttons_wrapper {
  height: 40px;
  float: right;
  margin-right: 20px;
}
.message_author {
  font-size: 14px;
  font-weight: bold;
  background-repeat: no-repeat;
  background-size: 100%;
  background-position: center;
  min-width: 30px;
  min-height: 30px;
  align-self: center;
  margin-right: 15px;
}
#chat_text {
  width: 290px;
  margin-bottom: -15px;
  height: 50px;
  border: none;
}
.message__container {
  padding-bottom: 10px;
  display: block;
}
.message__recieved {
  color: white;
  background-color: #4e8cff;
  max-width: calc(100% - 120px);
  word-wrap: break-word;
  margin: 24px;
}
</style>
