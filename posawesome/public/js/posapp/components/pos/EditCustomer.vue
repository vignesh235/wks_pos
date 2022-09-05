<template>
  <v-row justify="center">
    <v-dialog v-model="customerDialog" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline indigo--text">{{ __('Customer Info') }}</span>
        </v-card-title>
        <v-card-text class="pa-0">
          <v-container>
            <v-row>
              <v-col cols="6">
                <v-text-field
                  dense
                  color="indigo"
                  :label="frappe._('Customer Name')"
                  background-color="white"
                  hide-details
                  readonly
                  v-model="customer_info.customer_name"
                ></v-text-field>
              </v-col>
              <v-col cols="6">
                <v-autocomplete
                  dense
                  color="indigo"
                  :label="frappe._('Customer Group')"
                  background-color="white"
                  hide-details
                  v-model="customer_info.customer_group"
                  :items="groups"
                  @change="set_customer_info('customer_group', $event)"
                ></v-autocomplete>
                <!-- <v-autocomplete
                  dense
                  color="indigo"
                  :label="frappe._('Type')"
                  background-color="white"
                  hide-details
                  v-model="cus_bls_info.customer_type"
                  :items="types"
                  @change="set_cus_bls_info('customer_type', $event)"
                ></v-autocomplete> -->
              </v-col>
              <v-col cols="6">
                <v-text-field
                  dense
                  color="indigo"
                  :label="frappe._('Mobile No1')"
                  background-color="white"
                  hide-details
                  v-model="cus_bls_info.mobile_no"
                  @change="set_customer_info('mobile_no', $event)"
                ></v-text-field>
              </v-col>
               <!-- <v-col cols="6">
                <v-text-field
                  dense
                  color="indigo"
                  :label="frappe._('GSTIN')"
                  background-color="white"
                  hide-details
                  v-model="cus_bls_info.gstin"
                  @change="set_cus_bls_info('gstin', $event)"
                ></v-text-field>
              </v-col>
               <v-col cols="6">
                <v-text-field
                  dense
                  color="indigo"
                  :label="frappe._('PAN')"
                  background-color="white"
                  hide-details
                  v-model="cus_bls_info.pan"
                  @change="set_cus_bls_info('pan', $event)"
                ></v-text-field>
              </v-col> -->
              <v-col cols="6">
                <v-text-field
                  dense
                  color="indigo"
                  :label="frappe._('Address Line_1')"
                  background-color="white"
                  hide-details
                  v-model="cus_bls_info.address_line_1"
                  @change="set_customer_info('address_line_1', $event)"
                ></v-text-field>
              </v-col>
              <v-col cols="6">
                <v-text-field
                  dense
                  color="indigo"
                  :label="frappe._('Address Line_2')"
                  background-color="white"
                  hide-details
                  v-model="cus_bls_info.address_line_2"
                  @change="set_customer_info('address_line_2', $event)"
                ></v-text-field>
              </v-col>
               <v-col cols="6">
                <v-autocomplete
                  dense
                  color="indigo"
                  :label="frappe._('City')"
                  background-color="white"
                  hide-details
                  v-model="cus_bls_info.city"
                  :items="citys"
                  @change="('city', $event)"
                ></v-autocomplete> 
              </v-col>
             <v-col cols="6">
              <v-autocomplete
                  dense
                  color="indigo"
                  :label="frappe._('State')"
                  background-color="white"
                  hide-details
                  v-model="cus_bls_info.state"
                  :items="states"
                  @change="set_cus_bls_info('state', $event)"
                ></v-autocomplete>
              </v-col>
              <v-col cols="6">
                <v-autocomplete
                  dense
                  color="indigo"
                  :label="frappe._('Pin Code')"
                  background-color="white"
                  hide-details
                  v-model="cus_bls_info.pin_code_"
                  :items="pins"
                  @change="set_cus_bls_info('pin_code_', $event)"
                ></v-autocomplete>
              </v-col>
              <v-col cols="6">
              <v-autocomplete
                  dense
                  color="indigo"
                  :label="frappe._('Territory')"
                  background-color="white"
                  hide-details
                  v-model="cus_bls_info.territory"
                  :items="territorys"
                  @change="set_cus_bls_info('territory', $event)"
                ></v-autocomplete>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="error" dark @click="close_dialog">Close</v-btn>
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
    address_line_1:'',
    address_line_2:'',
    customer_info: '',
    vehicleDialog: false,
    vehicle_info:'',
    v_category: '',
    v_model: '',
    types:'',
    citys:'',
    groups:'',
    states:'',
    pins:'',
    cus_bls_info:'',
    mobile_no:'',
    territorys:'',
    
    
    


  }),

  // watch: {
  //   customer() {
  //     this.fetch_customer_details();
  //   },
  // },

  methods: {
    close_dialog() {
      this.customerDialog = false;
    },
     close__vehicle_dialog() {
      this.vehicleDialog = false;
    },
    set_customer_info(field, value) {
      const vm = this;
      console.log("adress")
      console.log(field,value)
      frappe.call({
        method: 'posawesome.posawesome.api.posapp.set_customer_info',
        args: {
          fieldname: field,
          customer: this.customer_info.name,
          value: value,
        },
        callback: (r) => {
          if (!r.exc) {
            vm.customer_info[field] = value;
            evntBus.$emit('show_mesage', {
              text: __('Customer contact edited successfully.'),
              color: 'success',
            });
            frappe.utils.play_sound('submit');
          }
        },
      });
    },



    set_cus_bls_info(field, value) {
      const vm = this;
      frappe.call({
        method: 'posawesome.posawesome.api.posapp.set_customer_info',
        args: {
          fieldname: field,
          customer: this.customer_info.name,
          value: value,
        },
        callback: (r) => {
          if (r.exc) {
            vm.customer_info[field] = value;
            evntBus.$emit('show_mesage', {
              text: __('Customer contact updated successfully.'),
              color: 'success',
            });
            frappe.utils.play_sound('submit');
          }
        },
      });
    },
  },



  created: function () {
    
    evntBus.$on('open_edit_customer', () => {
      this.customerDialog = true;
      var  groups, citys, types, states, pins,mobile_no,territorys;
      // v_model;
      frappe.call({
        method: "posawesome.posawesome.api.posapp.customer_link_details",
        async: false,
        callback(r){
        groups = r.message[0]
        citys= r.message[2]
        types=r.message[3]
        states=r.message[4]
        pins=r.message[5]
        mobile_no=r.message[6]
        territorys=r.message[1]

          }
      })
      this.groups = groups
      this.territorys=territorys
      this.citys= citys
      this.types=types
      this.states=states
      this.pins=pins
    });


    evntBus.$on('set_customer_info_to_edit', (data) => {
      this.customer_info = data;
      var cus_info;
      
      frappe.call({
        
        method: "posawesome.posawesome.api.posapp.customer_bls_details",
        async: false,
        args: {
						cus: data.customer_name,
					},
        callback(r){
          console.log("cus_info")
             cus_info = r.message[0]
             console.log(cus_info)
            
          }
      })
      
      this.cus_bls_info = cus_info; 
      
    });
  },
};
</script>



