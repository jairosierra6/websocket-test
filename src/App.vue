<template>
  <v-app id="inspire">

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
                  md="4"
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
                  md="4"
                >
                  <v-text-field
                    label="Middle name"
                    v-model="mName"
                    hint="example of helper text only on focus"
                  ></v-text-field>
                </v-col>
                <v-col
                  cols="12"
                  sm="6"
                  md="4"
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
        <v-avatar
          class="mb-4"
          color="grey darken-1"
          size="64"
        >
        
          <img v-if="userLoaded" src='https://i.pravatar.cc/300'/>

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
            <v-checkbox
                v-model="checkLanguage"
                label="Enable event languages"
                color="primary"
                value="language"
                hide-details
            ></v-checkbox>
            <v-checkbox
                v-model="checkLogin"
                label="Enable auto login"
                color="primary"
                value="auto login"
                hide-details
            ></v-checkbox>
            <v-checkbox
                v-model="checkVideo"
                label="Enable video"
                color="primary"
                value="video"
                hide-details
            ></v-checkbox>
            <v-text-field
                v-model="configuration"
                label="Name of the event"
                placeholder="Name of the event"
            ></v-text-field>
            <v-text-field
                v-model="url"
                label="Vep url of the event"
                placeholder="Url of the event"
            ></v-text-field>

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
      mName: '',
      lName: '',
      newEventData: '',
      nameRules: [
          (v) => !!v || 'Field is required'
        ],
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
      testIng(msg){
        console.log('AAAAA, ', msg);
      },
      sendUpdate() {
        const structure = {
          "action": "setEvent",
          "roomId": `${this.selectedEvent}`,
          "users": `{"name": "${this.fName} ${this.mName} ${this.lName}", "id": "${this.selectedEvent}"}` ,
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
        console.log('Name=====>>> ', message.name);
        switch (message.name) {
          case 'eventList':
            console.log('eventList: ', JSON.parse(message.data));
            _this.eventList = JSON.parse(message.data);
            _this.dialog = true;
            console.log('Names ====>> ', this.eventList.map(event => event.eventName));
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