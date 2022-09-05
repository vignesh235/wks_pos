
 <template>
  <v-row justify="center">
    <v-dialog v-model="customerDialog" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline primary--text">{{ __('New Customer') }}</span>
        </v-card-title>
        <v-card-text class="pa-0">
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  dense
                  autofocus
                  color="primary"
                  :label="frappe._('Customer Name')"
                  background-color="white"
                  hide-details
                  v-model="customer_name"
                ></v-text-field>
              </v-col>
              <v-col cols="6">
                <v-autocomplete
                  clearable
                  dense
                  auto-select-first
                  color="primary"
                  :label="frappe._('Customer Group')"
                  v-model="customer_group"
                  :items="groups"
                  background-color="white"
                  class="group"
                  @change="group($event)"
                  :no-data-text="__('Group not found')"
                  hide-details
                >
                </v-autocomplete>
              </v-col>
              <v-col cols="6">
                <v-text-field
                  dense
                  color="primary"
                  :label="frappe._('Mobile No')"
                  background-color="white"
                  hide-details
                  v-model="customer_mobile_no"
                ></v-text-field>
              </v-col>
              <!-- <v-col cols="6">
                <v-text-field
                  dense
                  color="primary"
                  :label="frappe._('GSTIN')"
                  background-color="white"
                  hide-details
                  v-model="customer_gst"
                ></v-text-field>
              </v-col> -->
              <v-col cols="6">
                <v-text-field
                  dense
                  color="primary"
                  :label="frappe._('Address1*')"
                  background-color="white"
                  hide-details
                  v-model="customer_address1"
                ></v-text-field>
              </v-col>
              <v-col cols="6">
                <!-- <v-menu
                  ref="birthday_menu"
                  v-model="birthday_menu"
                  :close-on-content-click="false"
                  transition="scale-transition"
                  dense
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="birthday"
                      :label="frappe._('Birthday')"
                      readonly
                      dense
                      clearable
                      hide-details
                      v-bind="attrs"
                      v-on="on"
                      color="primary"
                    ></v-text-field>
                  </template>
                  <v-date-picker
                    v-model="birthday"
                    color="primary"
                    no-title
                    scrollable
                    :max="frappe.datetime.now_date()"
                    @input="birthday_menu = false"
                  >
                  </v-date-picker>
                </v-menu> -->
                <v-text-field
                  dense
                  color="primary"
                  :label="frappe._('Address 2')"
                  background-color="white"
                  hide-details
                  v-model="customer_address2"
                ></v-text-field>
              </v-col>
              <v-col cols="6">
                <v-autocomplete
                  clearable
                  dense
                  background-color="white"
                  color="primary"
                  :label="frappe._('City')"
                  v-model="customer_city"
                  :items="city"
                 ></v-autocomplete>  
              </v-col>
              <v-col cols="6">
                <v-autocomplete
                  clearable
                  dense
                  auto-select-first
                  color="primary"
                  :label="frappe._('Territory')"
                  v-model="territory"
                  :items="territorys"
                  background-color="white"
                  :no-data-text="__('State not found')"
                  hide-details
                >
                </v-autocomplete>
              </v-col>
              <v-col cols="6">
                <v-autocomplete
                  clearable
                  dense
                  background-color="white"
                  color="primary"
                  :label="frappe._('Pincode')"
                  :items="pincode"
                  
                  v-model="customer_pincode"
                 ></v-autocomplete>
               <!-- <v-text-field
                  dense
                  color="primary"
                  :label="frappe._('state')"
                  background-color="white"
                  hide-details
                  v-model="customer_state"
                ></v-text-field> -->
              </v-col>
              <v-col cols="6">
                <v-autocomplete
                  clearable
                  dense
                  background-color="white"
                  color="primary"
                  :label="frappe._('State')"
                  :items="state"
                  
                  v-model="customer_state"
                 ></v-autocomplete>
              
              </v-col>
              <v-col cols="6" >
                <v-text-field
                  
                  dense
                  color="primary"
                  :label="frappe._('Commission Rate')"
                  background-color="white"
                  hide-details
                  v-model="commission_rate"
                  class="tax"
                  style="visibility: hidden;"
                ></v-text-field>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="error" dark @click="close_dialog">{{
            __('Close')
          }}</v-btn>
          <v-btn color="success" dark @click="submit_dialog">{{
            __('Submit')
          }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
import { evntBus } from '../../bus';
export default {
  data: () => ({
    customerDialog: false,
    pos_profile: '',
    customer_name: '',
    territory:'',
    customer_mobile_no:'',
    customer_gst:'',
    customer_address1:'',
    customer_address2:'',
    customer_city:'',
    customer_state:'',
    customer_pincode:'',
    commission_rate:'',  
    groups: [],
    territorys: [],
    state:[],
    city:[],
    pincode:[],

  }),
  watch: {},
  methods: {
    group(event)
    {
      console.log(event)
      var tax=document.getElementsByClassName("tax")[0];
      if(event=="MECHANIC")
      {
        var tax=document.getElementsByClassName("tax")[0];

        tax.style.visibility="visible";
        tax.style.visibility="visible";
        
      }
      else{
        tax.style.visibility="hidden";
      }
    },
    close_dialog() {
      this.customerDialog = false;
    },
    getCustomerGroups() {
      if (this.groups.length > 0) return;
      const vm = this;
      frappe.db
        .get_list('Customer Group', {
          fields: ['name'],
          page_length: 1000,
        })
        .then((data) => {
          if (data.length > 0) {
            data.forEach((el) => {
              vm.groups.push(el.name);
            });
          }
        });
    },
    getCustomerTerritorys() {
      if (this.territorys.length > 0) return;
      const vm = this;
      frappe.db
        .get_list('Territory', {
          fields: ['name'],
          page_length: 1000,
        })
        .then((data) => {
          if (data.length > 0) {
            data.forEach((el) => {
              vm.territorys.push(el.name);
            });
          }
        });
    },
    getCustomerState(){
      const vm = this;
      frappe.call({
          method: 'posawesome.posawesome.api.posapp.get_state',
          args: {},
          callback: function (r) {
            if (r.message) {
              r.message.forEach((element) => {
                vm.state.push(element.name);
              });
            }
          },
        });
    },

    getCustomerCity(){
      const vm = this;
      frappe.call({
          method: 'posawesome.posawesome.api.posapp.get_City',
          args: {},
          callback: function (r) {
            if (r.message) {
              r.message.forEach((element) => {
                vm.city.push(element.name);
              });
            }
          },
        });
    },
    get_pincode(){
      const vm = this;
      frappe.call({
          method: 'posawesome.posawesome.api.posapp.get_pincode',
          args: {},
          callback: function (r) {
            if (r.message) {
              
              console.log(r.message)
              r.message.forEach((element) => {
                vm.pincode.push(element.ts_pincode);
              });
            }
          },
        });
    },
    submit_dialog() {
      if (this.customer_name) {
        const vm = this;
        const args = {
          customer_name: this.customer_name,
          // company: this.pos_profile.company,
          // tax_id: this.tax_id,
          customer_mobile_no: this.customer_mobile_no,
          // customer_gst: this.customer_gst,
          customer_address1:this.customer_address1,
          customer_address2:this.customer_address2,
          customer_city:this.customer_city,
          commission_rate:this.commission_rate,
          // referral_code: this.referral_code,
          // birthday: this.birthday,
          customer_state:this.customer_state,
          customer_group: this.customer_group,
          territory: this.territory,
        };
        frappe.call({
          method: 'posawesome.posawesome.api.posapp.create_customer',
          args: args,
          callback: (r) => {
            if (!r.exc && r.message.name) {
              evntBus.$emit('show_mesage', {
                text: __('Customer contact created successfully.'),
                color: 'success',
              });
              args.name = r.message.name;
              frappe.utils.play_sound('submit');
              evntBus.$emit('add_customer_to_list', args);
              evntBus.$emit('set_customer', r.message.name)
              vm.customer_name = '';
              vm.customer_mobile_no = '';
              // vm.customer_gst = '';
              vm.customer_address1 = '';
              vm.customer_address2 = '';
              vm.customer_city = '';
              vm.customer_state = '';
              vm.customer_group = '';
              vm.commission_rate='';
              vm.customerDialog = false;
            }
          },
        });
        this.customerDialog = false;
      }
    },
  },
  created: function () {
    evntBus.$on('open_new_customer', () => {
      this.customerDialog = true;
    });
    evntBus.$on('register_pos_profile', (data) => {
      this.pos_profile = data.pos_profile;
    });
    this.getCustomerGroups();
    this.getCustomerTerritorys();
    this.getCustomerState();
    this.getCustomerCity();
    this.get_pincode();
  },
};
</script>
