<template>
  <div>
    
    <v-autocomplete
      dense
      clearable
      auto-select-first
      outlined
      color="indigo"
      :label="frappe._('Customer')"
      v-model="customer"
      :items="customers"
      item-text="customer_name"
      item-value="name"
      background-color="white"
      :no-data-text="__('Customer not found')"
      hide-details
      :filter="customFilter"
      :disabled="readonly"
      append-icon="mdi-plus"
      @click:append="new_customer"
      prepend-inner-icon="mdi-account-edit"
      @click:prepend-inner="edit_customer"
      @change= "get_vehicle_list"
    >
      <template v-slot:item="data">
        <template>
          <v-list-item-content>
            <v-list-item-title
              class="indigo--text subtitle-1"
              v-html="data.item.customer_name"
            ></v-list-item-title>
            <v-list-item-subtitle
              v-if="data.item.customer_name != data.item.name"
              v-html="`ID: ${data.item.name}`"
            ></v-list-item-subtitle>
            <!-- <v-list-item-subtitle
              v-if="data.item.tax_id"
              v-html="`TAX ID: ${data.item.tax_id}`"
            ></v-list-item-subtitle> -->
            <v-list-item-subtitle
              v-if="data.item.gstin"
              v-html="`GSTIN: ${data.item.gstin}`"
            ></v-list-item-subtitle>
            <v-list-item-subtitle
              v-if="data.item.mobile_no"
              v-html="`Mobile No: ${data.item.mobile_no}`"
            ></v-list-item-subtitle>
            <v-list-item-subtitle
              v-if="data.item.address_line_1"
              v-html="`Address Line1: ${data.item.address_line_1}`"
            ></v-list-item-subtitle>
          </v-list-item-content>
        </template>
      </template>
    </v-autocomplete>
  </div>
  
</template>



<script>
import { evntBus } from '../../bus';
export default {
  data: () => ({
    pos_profile: '',
    customers: [],
    customer: '',
    vehicle: '',
    readonly: false,
  }),

  methods: {
    get_customer_names() {
      const vm = this;
      if (vm.pos_profile.posa_local_storage && localStorage.customer_storage) {
        vm.customers = JSON.parse(localStorage.getItem('customer_storage'));
      }
      frappe.call({
        method: 'posawesome.posawesome.api.posapp.get_customer_names',
        args: {
          pos_profile: this.pos_profile.pos_profile,
        },
        callback: function (r) {
          if (r.message) {
            vm.customers = r.message;
            if (vm.pos_profile.posa_local_storage) {
              localStorage.setItem('customer_storage', '');
              localStorage.setItem(
                'customer_storage',
                JSON.stringify(r.message)
              );
            }
          }
        },
      });
    },
    new_customer() {
      evntBus.$emit('open_new_customer');
      console.log(data.item.tax_id);
    },
    edit_customer() {
      evntBus.$emit('open_edit_customer');
    },
 
  
    customFilter(item, queryText, itemText) {
      const textOne = item.customer_name
        ? item.customer_name.toLowerCase().replace(/[^a-zA-Z0-9]/g, '') : '';
      const textTwo = item.tax_id ? item.tax_id.toLowerCase().replace(/[^a-zA-Z0-9]/g, '') : '';
      const textThree = item.email_id ? item.email_id.toLowerCase().replace(/[^a-zA-Z0-9]/g, '') : '';
      const textFour = item.mobile_no ? item.mobile_no.toLowerCase().replace(/[^a-zA-Z0-9]/g, '') : '';
      const textFifth = item.name.toLowerCase().replace(/[^a-zA-Z0-9]/g, '');
      const searchText = queryText.toLowerCase().replace(/[^a-zA-Z0-9]/g, '');

      return (
        textOne.indexOf(searchText) > -1 ||
        textTwo.indexOf(searchText) > -1 ||
        textThree.indexOf(searchText) > -1 ||
        textFour.indexOf(searchText) > -1 ||
        textFifth.indexOf(searchText) > -1
      );
    },
  },

  computed: {},

// CREATE
   created: function () {
    this.customer = customer;
    this.vehicle = customer; 
    var vm = this;
    var customer = vm.customer;
    var _h = vm.$createElement;
    var _c = vm._self._c || _h;
    var choosed_vehicle;
    var  credit_limit;
    
    this.$nextTick(function () {
      evntBus.$on('register_pos_profile', (pos_profile) => {
        this.pos_profile = pos_profile;
        this.get_customer_names();
      });
      evntBus.$on('set_customer', (customer) => {
        this.customer = customer;
        

      //.....................................................

        customer =vm.customer;
       var vehicle_details_dict = []
        var vehicle_list =[]
         if(customer){
         // this.get_vehicle_list();

          frappe.db.get_value("Customer",customer,'payment_terms').then(function(value){
            credit_limit = value.message.payment_terms
          })

        frappe.call({
          method : "posawesome.posawesome.api.credit_sales.customer_credit_sale",
          args:{
            customer: customer
          },
          callback : function(r){
            if(r.message[2]>0){
              let modes=[];
              for(var i=0; i<vm.pos_profile.pos_profile.payments.length; i++){
                modes.push(vm.pos_profile.pos_profile.payments[i].mode_of_payment)
              }
            var d = new frappe.ui.Dialog({
              title:"Customer: "+ customer +"'s Outstanding Amount",
              fields:[
                {'fieldname':'alert','fieldtype':'HTML','read_only':1,'bold':1},
                {'label':'Outstanding Amount','fieldname':'outstanding','fieldtype':'Currency','default':r.message[2],'read_only':1},
                {'label':'Paid Amount','fieldname':'amount','fieldtype':'Currency','reqd':1},
                {
                  'label':'Mode of Payment',
                  'fieldname':'mode',
                  'fieldtype':'Select',
                  'options':modes
                },
                {'label':'Reference Number','fieldname':'ref_no','fieldtype':'Data'},
                {'label':'Reference Date','fieldname':'ref_date','fieldtype':'Date'}
              ],
              primary_action : function(data){
                frappe.call(
                  'posawesome.posawesome.api.posapp.check_opening_shift', 
                  {user: frappe.session.user,}).then(function (id) {
                    if (r.message && data.amount && data.mode) {
                      frappe.call({
                        method:"posawesome.posawesome.api.credit_sales.payment_entry",
                        args:{
                        // amount,mode,customer,pending_invoice
      
                          amount:data.amount,
                          mode:data.mode,
                          customer:customer,
                          pending_invoice:r.message[1],
                          company:vm._data.pos_profile.company.company_name,
                          opening : id.message.pos_opening_shift.name,
                          ref_no: data.ref_no,
                          ref_date: data.ref_date
                          
                        },
                        callback : function(res){
                            if(res.message[0]){
                              evntBus.$emit('show_mesage', {
                                text: __('Payment Entry Created Successfully.'),
                                color: 'success',
                              });
                              var mode = r.message[1]
                              var payment = r.message[0]

                            }
                        }
                      });
                    }
                  });

                
                d.hide();
              }
            });
            var template = r.message[3]
            d.set_df_property('alert','options',frappe.render(template,{}))
            d.show();
          }
        }
        });
        }

        //...................................................
 
      return _c(
         _c("v-autocomplete", {
          attrs: {
            dense: "",
            clearable: "",
            "auto-select-first": "",
            outlined: "",
            color: "indigo",
            label: vm.frappe._("Vehicle"),
            items: vehicle_details_dict,
            "item-text": "name",
            "item-value": "name",
            "background-color": "white",
            "no-data-text": vm.__("Vehicle not found"),
            "hide-details": "",
            filter: vm.customFilterVehicle,
            disabled: vm.readonly,
            "append-icon": "mdi-plus",
            "prepend-inner-icon": "mdi-account-edit"
          },
          on: {
            "click:append": vm.new_vehicle,
            "click:prepend-inner": vm.edit_vehicle,
            change: function($$v) {
              vm.get_vehicle_list()
               choosed_vehicle = $$v;
              vehicle_details_dict.forEach(function(r){
                if(r.name === choosed_vehicle){
                  vehicle_info = r
                  return ''
                }
              })
            },
          },
          scopedSlots: vm._u([
            {
              key: "item",
              fn: function(data) {
                return [
                  [
                    _c(
                      "v-list-item-content",
                      [
                        _c("v-list-item-title", {
                          staticClass: "indigo--text subtitle-1",
                          domProps: { innerHTML: vm._s(data.item.name) }
                        }),
                        vm._v(" "),
                        data.item.customer_name != data.item.name
                          ? _c("v-list-item-subtitle", {
                              domProps: {
                                innerHTML: vm._s("ID: " + data.item.name),
                              },domProps: {
                                innerHTML: vm._s("ID: " + data.item.name),
                              }
                            })
                          : vm._e(),
                        vm._v(" "),
                        data.item.vehicle_category
                          ? _c("v-list-item-subtitle", {
                              domProps: {
                                innerHTML: vm._s("Category: " + data.item.vehicle_category)
                              }
                            })
                          : vm._e(),
                        vm._v(" "),
                        data.item.ts_vehicle_model
                          ? _c("v-list-item-subtitle", {
                              domProps: {
                                innerHTML: vm._s("Model: " + data.item.ts_vehicle_model)
                              }
                            })
                          : vm._e(),
                        vm._v(" "),
                        data.item.vehicle_year
                          ? _c("v-list-item-subtitle", {
                              domProps: {
                                innerHTML: vm._s(
                                  "Year: " + data.item.vehicle_year
                                )
                              }
                            })
                          : vm._e()
                      ],
                      1
                    )
                  ]
                ]
              }
            }
          ]),
          model: {
            value: choosed_vehicle,
            callback: function($$v) {
              choosed_vehicle = $$v;
              vm.choosed_vehicle = $$v
            },
            expression: "choosed_vehicle"
          }
         
        })

      )  



        //.............................................................
      });
      evntBus.$on('add_customer_to_list', (customer) => {
        this.customers.push(customer);
      });
      evntBus.$on('set_customer_readonly', (value) => {
        this.readonly = value;
      });
    });
  },

  watch: {
    customer() {
      evntBus.$emit('update_customer', this.customer);
    },
   
  },
};
</script>
