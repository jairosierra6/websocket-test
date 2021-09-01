<template>
  <v-app id="inspire">
    <v-system-bar height="50" app>
      <h1><b>Event:</b> {{eventTitle}}</h1>
      <v-spacer></v-spacer>

      <v-tooltip v-for="user in usersOnThisEvent" :key="user.connectionId" bottom>
      <template v-slot:activator="{ on, attrs }">
        <v-avatar :color="user.id" size="40" v-bind="attrs" v-on="on" style="margin: 2px">
          <span style="color: black; font-weight: bold;">{{user.firstName.toUpperCase().charAt(0)}}{{user.lastName.toUpperCase().charAt(0)}}</span>
        </v-avatar>
      </template>
      <span>{{user.firstName}} {{user.lastName}}</span>
    </v-tooltip>     
    </v-system-bar>
    <v-dialog
      v-model="standby"
      hide-overlay
      persistent
      width="300"
    >
      <v-card
        color="primary"
        dark
      >
        <v-card-text>
          Please stand by
          <v-progress-linear
            indeterminate
            color="white"
            class="mb-0"
          ></v-progress-linear>
        </v-card-text>
      </v-card>
    </v-dialog>
    <div>
      <v-dialog
        v-model="dialog"
        persistent
        max-width="600px"
      >
        <v-card>
          <v-card-title>
            <span class="text-h5">Connect to event</span>
          </v-card-title>
          <v-card-text>
            <v-container>
              <v-row>
                <v-col
                  cols="12"
                  sm="6"
                >
                  <v-text-field
                    label="First name*"
                    v-model="fName"
                    :rules="nameRules"
                    required
                  ></v-text-field>
                </v-col>
                <v-col
                  cols="12"
                  sm="6"
                >
                  <v-text-field
                    label="Last name*"
                    v-model="lName"
                    :rules="nameRules"
                    hint="example of persistent helper text"
                    persistent-hint
                    required
                  ></v-text-field>
                </v-col>
                <v-col
                  cols="12"
                  sm="6"
                >
                  <v-select v-if="eventList !== undefined"
                    v-model="selectedEvent"
                    :items="eventList"
                    :item-text="'eventName'"
                    :item-value="'roomId'"
                    label="Select your event*"
                    :rules="nameRules"
                    required
                  ></v-select>
                </v-col>
              </v-row>
            </v-container>
            <small>*indicates required field</small>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="blue darken-1"
              text
              @click="sendUpdate()"
            >
              Connect
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>    

    <v-navigation-drawer
      v-model="drawer"
      app
    >
      <v-sheet
        :color="jsonData.isDark ? 'grey darken-4': 'grey lighten-4'"
        class="pa-4"
      >
        <v-avatar class="mb-4" :color="this.color" size="64">
          <span style="color: black; font-weight: bold; font-size: 35px;">{{fName.toUpperCase().charAt(0)}}{{lName.toUpperCase().charAt(0)}}</span>        
        </v-avatar>

        <div >{{username}}</div>
      </v-sheet>

      <v-divider></v-divider>

      <v-list>
        <v-list-item
          v-for="[icon, text] in links"
          :key="icon"
          link
          v-on:click="disconnect()"
        >
          <v-list-item-icon>
            <v-icon>{{ icon }}</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>{{ text }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item>
          <v-switch
            id="jsonData.isDark"
            v-model="jsonData.isDark"
            :label="`Dark Theme`"
            @change="changeTheme(jsonData.isDark)"
          ></v-switch>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <v-container
        class="py-8 px-6"
        fluid
      >
        <v-row>
          <v-col cols="12">

          <!-- Controls -->

          <!--  -->
          <v-card
            class="mx-auto"
            outlined
          >
            <v-list-item three-line>
              <v-list-item-content>
                <v-list-item-title class="text-h5 mb-1">
                  <v-checkbox
                    id="jsonData.checkLanguage"
                    v-model="jsonData.checkLanguage"
                    label="Enable event languages"
                    color="primary"
                    hide-details
                    @change="sendChangedData(jsonData.checkLanguage, 'checkLanguage')"
                    v-on:focus="sendCurrentLocation()"
                ></v-checkbox>
                </v-list-item-title>
                <v-list-item-subtitle>Description of this option</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
        
            <v-card-actions>
              {{usersEditing.checkLanguage}}
            </v-card-actions>
          </v-card>
          <!--  -->
          <v-card
            class="mx-auto"
            outlined
          >
            <v-list-item three-line>
              <v-list-item-content>
                <v-list-item-title class="text-h5 mb-1">
                  <v-checkbox
                    id="jsonData.checkLogin"
                    v-model="jsonData.checkLogin"
                    label="Enable auto login"
                    color="primary"
                    hide-details
                    @change="sendChangedData(jsonData.checkLogin, 'checkLogin')"
                    v-on:focus="sendCurrentLocation()"
                ></v-checkbox>
                </v-list-item-title>
                <v-list-item-subtitle>Description of this option</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
        
            <v-card-actions>
              {{usersEditing.checkLogin}}
            </v-card-actions>
          </v-card>
          <!--  -->
          <v-card
            class="mx-auto"
            outlined
          >
            <v-list-item three-line>
              <v-list-item-content>
                <v-list-item-title class="text-h5 mb-1">
                  <v-checkbox
                    id="jsonData.checkVideo"
                      v-model="jsonData.checkVideo"
                      label="Enable video"
                      color="primary"
                      hide-details
                      @change="sendChangedData(jsonData.checkVideo, 'checkVideo')"
                      @mouseover="sendCurrentLocation()"
                  ></v-checkbox>
                </v-list-item-title>
                <v-list-item-subtitle>Description of this option</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
        
            <v-card-actions>
              {{usersEditing.checkVideo}}
            </v-card-actions>
          </v-card>
          <!--  -->
          <v-card
            class="mx-auto"
            outlined
          >
            <v-list-item three-line>
              <v-list-item-content>
                <v-list-item-title class="text-h5 mb-1">
                  <v-text-field
                      id="jsonData.eventName"
                      v-model="jsonData.eventName"
                      label="Name of the event"
                      placeholder="Name of the event"
                      v-on:blur="sendChangedData(jsonData.eventName, 'eventName')"
                      v-on:focus="sendCurrentLocation()"
                  ></v-text-field>
                </v-list-item-title>
                <v-list-item-subtitle>Description of this option</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
        
            <v-card-actions>
              <!-- convert to a component -->
                <v-tooltip v-for="user in usersEditing.eventName" :key="user.connectionId" bottom>
                  <template v-slot:activator="{ on, attrs }">
                    <v-avatar class="pulse-animation" :color="user.id" size="25" v-bind="attrs" v-on="on" style="margin: 2px">
                      <span style="color: black; font-weight: bold; font-size: 10px;">{{user.firstName.toUpperCase().charAt(0)}}{{user.lastName.toUpperCase().charAt(0)}}</span>
                    </v-avatar>
                  </template>
                  <span>{{user.firstName}} {{user.lastName}}</span>
                </v-tooltip>  
              <!-- convert to a component -->      
            </v-card-actions>
          </v-card>
          <!--  -->
          <v-card
            class="mx-auto"
            outlined
          >
            <v-list-item three-line>
              <v-list-item-content>
                <v-list-item-title class="text-h5 mb-1">
                  <v-text-field
                    id="jsonData.url"
                    v-model="jsonData.url"
                    label="Vep url of the event"
                    placeholder="Url of the event"
                    v-on:blur="sendChangedData(jsonData.url, 'url')"
                    v-on:focus="sendCurrentLocation()"
                ></v-text-field>
                </v-list-item-title>
                <v-list-item-subtitle>Description of this option</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
        
            <v-card-actions>
              <!-- convert to a component -->
                <v-tooltip v-for="user in usersEditing.url" :key="user.connectionId" bottom>
                  <template v-slot:activator="{ on, attrs }">
                    <v-avatar class="pulse-animation" :color="user.id" size="25" v-bind="attrs" v-on="on" style="margin: 2px">
                      <span style="color: black; font-weight: bold; font-size: 10px;">{{user.firstName.toUpperCase().charAt(0)}}{{user.lastName.toUpperCase().charAt(0)}}</span>
                    </v-avatar>
                  </template>
                  <span>{{user.firstName}} {{user.lastName}}</span>
                </v-tooltip>  
              <!-- convert to a component -->     
            </v-card-actions>
          </v-card>
          <!-- Controls -->
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>
<style scoped>
.pulse-animation {
  border:1px solid white;    
    border-radius:50%;
    -moz-border-radius:50%;
    -webkit-border-radius:50%;
  animation: pulse 0.4s ease infinite alternate;
}
@keyframes pulse {
  0% {
    transform: scale(1);
    box-shadow: 0 2px 2px rgba(0,0,0,.2);
    border-radius: 50%;
  }
  100% {
    transform: scale(1.15);
    box-shadow: 0 2px 2px rgba(0,0,0,.2);
    border-radius: 45%;
  }
}
</style>
<script>
  export default {
    data: () => ({
      standby: true,
      connection: null,
      socketUrl: 'wss://tnpbcownl8.execute-api.us-east-2.amazonaws.com/dev',
      username: 'User Name',
      userLoaded: false,
      dialog: false,
      cards: ['Category 1', 'Category 2'],
      drawer: null,
      links: [
        ['mdi-delete', 'Disconnect'],
        ['mdi-alert-octagon', 'Status'],
      ],
      eventList: [],
      selectedEvent: {},
      fName: '',
      lName: '',
      newEventData: '',
      nameRules: [
          (v) => !!v || 'Field is required'
        ],
      usersOnThisEvent: [],
      eventTitle: '',
      color: '',
      checkLanguage: '',
      checkLogin: '',
      checkVideo: '',
      configuration1: '',
      url: '',
      jsonData: {},
      usersEditing: {},
    }),
    watch: {
      eventList: function() {
        console.log('Event changed ', this.eventList);
      },
      jsonData: function(value) {
        console.log('jsonData changed ', this.jsonData, '\nValue: ', value);
      }
    },
    methods: {
      changeTheme(value){
        this.toggle(value);
        this.sendChangedData(value, 'isDark')
        this.$forceUpdate();
      },
      toggle(value) {
        this.$vuetify.theme.dark = value;
        console.log('VALUE ', value, '\n\nTYPE: ', typeof(value));
      },
      sendCurrentLocation() {
        console.log('\nTHISSS ',event.target.id);
        const structure = {
            "action": "locatedOn",
            "roomId": `${this.selectedEvent}`,
            "route": `${event.target.id}`,
          }
          console.log('STRUCTURE ', structure);
          this.sendMessage(JSON.stringify(structure));
      },
      sendChangedData(dataToSend, name) {
        if (dataToSend !== null && dataToSend !== undefined) {
          let test = Object.keys(this.jsonData);
          console.log('test ', test, '\nTST2 ', Object.keys(dataToSend));
          const structure = {
            "action": "changedData",
            "roomId": `${this.selectedEvent}`,
            "route": `${name}`,
            "newData": `${dataToSend}`,
          }
          console.log('STRUCTURE ', structure);
          this.sendMessage(JSON.stringify(structure));
        }
      },
      sendMessage(message) {
        console.log('CONNECTION: ', this.connection);
        this.connection.send(message);
      },
      getRandomColor() {
        var letters = '0123456789ABCDEF';
        var color = '#';
        for (var i = 0; i < 6; i++) {
          color += letters[Math.floor(Math.random() * 16)];
        }
        return color;
      },
      testIng(msg){
        console.log('AAAAA, ', msg);
      },
      disconnect() {
        const structure = {
          "action": "$disconnect"
          }
          this.sendMessage(JSON.stringify(structure));
          location.reload();
      },
      sendUpdate() {
        this.eventTitle = this.eventList.find( x => x.roomId == this.selectedEvent).eventName;
        console.log('eventTitle: ', this.eventTitle);
        this.username = `${this.fName} ${this.lName}`;
        this.color = this.getRandomColor();
        const structure = {
          "action": "setEvent",
          "roomId": `${this.selectedEvent}`,
          "users": `{"name": "${this.fName}+${this.lName}", "id": "${this.color}"}` ,
          "eventJSON":null
        }
        console.log('STRUCTURE: ', JSON.stringify(structure));
        this.sendMessage(JSON.stringify(structure));
        this.dialog = false;
      },
      inactivity() {
        let r = confirm("Connection closed due to activity, do you want to reload?");
        if (r == true) {
          location.reload();
        } 
      },
    },
    created: function() {
      console.log("Starting connection to WebSocket Server")
      this.connection = new WebSocket(this.socketUrl);
      this.jsonData.isDark =true;
      const _this = this;

      this.connection.onmessage = function(event) {
        //console.log('EVENTTTT ', JSON.parse(event.data));
        let message = JSON.parse(event.data);
        console.log('MESGGG ', message);
        console.log('Name=====>>> ', message.name);
        switch (message.name) {
          case 'eventList':
            console.log('eventList: ', JSON.parse(message.data));
            _this.eventList = JSON.parse(message.data);
            _this.standby = false;
            _this.dialog = true;
            try {
              console.log('Event Names ====>> ', this.eventList.map(event => event.eventName));              
            } catch (error) {
              console.log('ERROR> ', error);
            }
            break;
            case 'updatedEvent':
            console.log('NEW EVENT DATA: ', JSON.parse(message.data));
            // alert the new event data
            break;
            case 'connectedUser':
              // eslint-disable-next-line no-case-declarations
              // eslint-disable-next-line no-case-declarations
              const temp = {
                connectionId: message.data.connectionId,
                id: message.data.id,
                firstName: message.data.name.split('+')[0],
                lastName: message.data.name.split('+')[1]
              }
              _this.usersOnThisEvent.push(temp);
              console.log('NEW USER CONNECTED: ', message.data);
            break;
            case 'connectedUsers':
              // eslint-disable-next-line no-case-declarations
              //_this.usersOnThisEvent.push(temp3);
              console.log('USER LIST CONNECTED: ', message.data);
              // eslint-disable-next-line no-case-declarations
              let response = message.data.connectedUsers;
              response.forEach(element => {
                element = JSON.parse(element);
                const tempo = {
                  connectionId: element.connectionId,
                  id: element.id,
                  firstName: element.name.split('+')[0],
                  lastName: element.name.split('+')[1]
                }
              _this.usersOnThisEvent.push(tempo);
              });
            break;
            case 'disconnectedUser':
              // eslint-disable-next-line no-case-declarations
              let disconnectedUser = message.data;
              console.log('DISCONNECTED USER: ', disconnectedUser);
              console.log('BEFORE DISCON ', _this.usersOnThisEvent);
              _this.usersOnThisEvent = _this.usersOnThisEvent.filter(element => element.connectionId !== disconnectedUser.connectionId)
              console.log('AFTER DISCON ', _this.usersOnThisEvent);
            break;
            case 'changedData':
              // eslint-disable-next-line no-case-declarations
              switch (message.route) {
                case 'eventName':
                  _this.jsonData.eventName = message.newData;
                  try {
                    _this.usersEditing = _this.usersEditing.eventName.filter(x => x.connectionId !== message.connectionId)
                  } catch (error) {console.log(error);}
                  break;
                case 'url':
                  _this.jsonData.url = message.newData;
                  try {
                    _this.usersEditing = _this.usersEditing.url.filter(x => x.connectionId !== message.connectionId)
                  } catch (error) {console.log(error);}
                  break;
                case 'checkLanguage':
                  _this.jsonData.checkLanguage = (message.newData === 'true') ? true : false;
                  try {
                    _this.usersEditing = _this.usersEditing.checkLanguage.filter(x => x.connectionId !== message.connectionId)
                  } catch (error) {console.log(error);}
                  break;
                case 'checkLogin':
                  _this.jsonData.checkLogin = (message.newData === 'true') ? true : false;
                  try {
                    _this.usersEditing = _this.usersEditing.checkLogin.filter(x => x.connectionId !== message.connectionId)
                  } catch (error) {console.log(error);}
                  break;
                case 'checkVideo':
                  _this.jsonData.checkVideo = (message.newData === 'true') ? true : false;
                  try {
                    _this.usersEditing = _this.usersEditing.checkVideo.filter(x => x.connectionId !== message.connectionId)
                  } catch (error) {console.log(error);}
                  
                  break;
                case 'isDark':
                  // eslint-disable-next-line no-case-declarations
                  let result = (message.newData === 'true') ? true : false;
                  _this.jsonData.isDark = result;
                  _this.toggle(result);
                  break;
                default:
                  break;
              }
              _this.$forceUpdate();            
            break;
            case 'locatedOn':
              // eslint-disable-next-line no-case-declarations
              let userToAdd = {};
              if (message.route){
                userToAdd = _this.usersOnThisEvent.find( x => x.connectionId == message.connectionId );
                console.log('\nUSRSS:>>>>>> ', _this.usersEditing);
              }
              switch (message.route) {
                case 'jsonData.eventName':
                  if (_this.usersEditing.eventName === undefined) {
                    _this.usersEditing.eventName = []
                  }
                  _this.usersEditing.eventName.push(userToAdd);                  
                  break;
                case 'jsonData.url':
                  if (_this.usersEditing.url === undefined) {
                    _this.usersEditing.url = []
                  }
                  _this.usersEditing.url.push(userToAdd); 
                  break;
                case 'jsonData.checkLanguage':
                  if (_this.usersEditing.checkLanguage === undefined) {
                    _this.usersEditing.checkLanguage = []
                  }
                  _this.usersEditing.checkLanguage.push(userToAdd); 
                  break;
                case 'jsonData.checkLogin':
                  if (_this.usersEditing.checkLogin === undefined) {
                    _this.usersEditing.checkLogin = []
                  }
                  _this.usersEditing.checkLogin.push(userToAdd); 
                  break;
                case 'jsonData.checkVideo':
                  if (_this.usersEditing.checkVideo === undefined) {
                    _this.usersEditing.checkVideo = []
                  }
                  _this.usersEditing.checkVideo.push(userToAdd); 
                  break;
                case 'jsonData.isDark':
                  break;
              
                default:
                  break;
              }
              _this.$forceUpdate();   
            break;
        
          default:
            console.log('Out of bound event', message);
            break;
        }
        console.log(event);
      }
      this.connection.onopen = function(event) {
        console.log(event)
        console.log("Successfully connected to the websocket server...")
       _this.connection.send('{"action": "listEvents"}');
      }
      this.connection.onclose = function(event) {
        console.log('closed connection: ', event);
        _this.inactivity();
      }
      this.connection.onerror = function(event) {
        console.log('error: ', event);
      }


    }
  }
</script>