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
        color="grey darken-4"
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
        >
          <v-list-item-icon>
            <v-icon>{{ icon }}</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>{{ text }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <v-container
        class="py-8 px-6"
        fluid
      >
        <v-row>
          <v-col
            v-for="card in cards"
            :key="card"
            cols="12"
          >

          <!-- Viejo ramiro desde aca puedes agregar los controles -->


          <!-- hasta aca agregar los controles -->
            <v-card>
              <v-subheader>{{ card }}</v-subheader>

              <v-list two-line>
                <template v-for="n in 3">
                  <v-list-item

                    :key="n"
                  >
                    <v-list-item-avatar color="grey darken-1">
                    </v-list-item-avatar>

                    <v-list-item-content>
                      <v-list-item-title>Message {{ n }}</v-list-item-title>

                      <v-list-item-subtitle>
                        Lorem ipsum dolor sit amet, consectetur adipisicing elit. Nihil repellendus distinctio similique
                      </v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>

                  <v-divider
                    v-if="n !== 3"
                    :key="`divider-${n}`"
                    inset
                  ></v-divider>
                </template>
              </v-list>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
  export default {
    data: () => ({
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
      color: ''
    }),
    watch: {
      eventList: function() {
        console.log('Event changed ', this.eventList);
      }
    },
    methods: {
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
      }
    },
    created: function() {
      console.log("Starting connection to WebSocket Server")
      this.connection = new WebSocket(this.socketUrl);
      
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
            _this.dialog = true;
            console.log('Event Names ====>> ', this.eventList.map(event => event.eventName));
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
              // eslint-disable-next-line no-case-declarations
              const tempo = {
                connectionId: message.data.connectionId,
                id: message.data.id,
                firstName: message.data.name.split('+')[0],
                lastName: message.data.name.split('+')[1]
              }
              _this.usersOnThisEvent.push(tempo);
              console.log('NEW USER CONNECTED: ', message.data);
            break;
        
          default:
            console.log('Out of bound event', event);
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
      }
      this.connection.onerror = function(event) {
        console.log('error: ', event);
      }


    }
  }
</script>