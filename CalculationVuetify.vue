<template>
  <div>
    <v-card v-for="(productList, k) in productList" :key="k">
      <v-divider class="mt-4"></v-divider>
      <v-card-title primary-title>Invoice Details</v-card-title>
      <v-card-text>
        <v-row>
          <v-col cols="12" md="2">
            <v-text-field
              label="Description"
              :rules="RequiredRule"
              v-model="productList.description"
              required
            ></v-text-field>
          </v-col>

          <v-col cols="12" md="2">
            <v-text-field
              label="HSN/SAC"
              :rules="IntegerNumberRule"
              v-model.number="productList.hsn_sac"
              v-on:keypress="isNumber($event)"
              required
            ></v-text-field>
          </v-col>

          <v-col cols="12" md="2">
            <v-text-field
              label="Quantity"
              v-on:keypress="isNumber($event)"
              v-model.number="productList.quantity"
              v-on:keyup="calculateLineTotal(productList)"
            ></v-text-field>
          </v-col>

          <v-col cols="12" md="2">
            <v-text-field
              label="Unit"
              :rules="RequiredRule"
              v-model.number="productList.unit"
              required
            ></v-text-field>
          </v-col>

          <v-col cols="12" md="2">
            <v-text-field
              label="Rate"
              v-model.number="productList.rate"
              v-on:keyup="calculateLineTotal(productList)"
            ></v-text-field>
          </v-col>
          <v-col cols="12" md="2">
            <v-text-field label="Amount" v-model.number="productList.amount" readonly></v-text-field>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="2">
            <v-select
              :items="SGSTCGSTRate"
              label="CGST Rate(%)"
              :options="reverse_charge_applicable"
              v-model="productList.cgst_rate"
              @change="calculateLineTotal(productList)"
            ></v-select>
          </v-col>

          <v-col cols="12" md="2">
            <v-text-field label="CGST Amount" v-model.number="productList.cgst_amount" readonly></v-text-field>
          </v-col>

          <v-col cols="12" md="2">
            <v-select
              :items="SGSTCGSTRate"
              label="SGST Rate(%)"
              :options="reverse_charge_applicable"
              v-model="productList.sgst_rate"
              @change="calculateLineTotal(productList)"
            ></v-select>
          </v-col>
          <v-col cols="12" md="2">
            <v-text-field label="SGST Amount" v-model.number="productList.sgst_amount" readonly></v-text-field>
          </v-col>
          <v-col cols="12" md="2">
            <v-select
              :items="IGSTRate"
              label="IGST Rate(%)"
              :options="reverse_charge_applicable"
              v-model="productList.igst_rate"
              @change="calculateLineTotal(productList)"
            ></v-select>
          </v-col>
          <v-col cols="12" md="2">
            <v-text-field label="IGST Amount" v-model.number="productList.igst_amount" readonly></v-text-field>
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12" md="2">
            <v-text-field v-model.number="productList.total_amount" label="Total Amount" readonly></v-text-field>
          </v-col>

          <v-col cols="12" md="2">
            <v-btn @click="deleteRow(k, productList)" absolute dark fab right color="red">
              <v-icon>mdi-delete</v-icon>
            </v-btn>
          </v-col>
        </v-row>
      </v-card-text>
      <!-- <v-divider class="mt-12"></v-divider> -->
    </v-card>
    <div>
      <v-btn @click="addNewRow" small color="primary">Add</v-btn>
    </div>
    <v-divider class="mt-6"></v-divider>

    <v-card>
      <v-card-title primary-title>Product Calculation</v-card-title>
      <v-card-text>
        <v-row no-gutters>
          <v-col :cols="5">
            <v-card class="pa-2" outlined tile>Amount Without Tax(Rs.)</v-card>
          </v-col>
          <v-col>
            <v-card class="pa-2" outlined tile>{{TotalAmountWithoutTax}}</v-card>
          </v-col>
        </v-row>

        <v-row no-gutters>
          <v-col :cols="5">
            <v-card class="pa-2" outlined tile>CGST Amount(Rs.)</v-card>
          </v-col>
          <v-col>
            <v-card class="pa-2" outlined tile>{{CGSTTotal}}</v-card>
          </v-col>
        </v-row>

        <v-row no-gutters>
          <v-col :cols="5">
            <v-card class="pa-2" outlined tile>SGST Amount(Rs.)</v-card>
          </v-col>
          <v-col>
            <v-card class="pa-2" outlined tile>{{SGSTTotal}}</v-card>
          </v-col>
        </v-row>

        <v-row no-gutters>
          <v-col :cols="5">
            <v-card class="pa-2" outlined tile>IGST Amount(Rs.)</v-card>
          </v-col>
          <v-col>
            <v-card class="pa-2" outlined tile>{{IGSTTotal}}</v-card>
          </v-col>
        </v-row>

        <v-row no-gutters>
          <v-col :cols="5">
            <v-card class="pa-2" outlined tile>Total Amount With Tax(Rs.)</v-card>
          </v-col>
          <v-col>
            <v-card class="pa-2" outlined tile>{{TotalAmountWithTax}}</v-card>
          </v-col>
        </v-row>

        <v-row no-gutters>
          <v-col :cols="5">
            <v-card class="pa-2" outlined tile>Amount in Words(INR)</v-card>
          </v-col>
          <v-col>
            <v-card class="pa-2" outlined tile>{{AmountinWords}}</v-card>
          </v-col>
        </v-row>
      </v-card-text>
    </v-card>

    <v-card class="mx-auto" max-width="344">
      <v-card-text>
        <div>{{invoice_subtotal}}</div>
        <br />
        <div>{{invoice_tax}}</div>
        <br />
        <div>{{invoice_total}}</div>
        <br />
        <div>{{productList}}</div>
      </v-card-text>
      <v-card-actions>
        <v-btn text color="deep-purple accent-4">Learn More</v-btn>
      </v-card-actions>
    </v-card>
  </div>
</template>


<script>
export default {
  data: () => ({
    TotalAmountWithoutTax: 0,
    CGSTTotal: 0,
    SGSTTotal: 0,
    IGSTTotal: 0,
    TotalAmountWithTax: 0,
    AmountinWords:"",
    SGSTCGSTRate: [2.5, 6, 9, 14],
    IGSTRate: [5, 12, 18, 28],
    productList: [
      {
        description: "",
        hsn_sac: "",
        quantity: "",
        unit: "",
        rate: "",
        amount: "",
        cgst_rate: 0,
        cgst_amount: "",
        sgst_rate: 0,
        sgst_amount: "",
        igst_rate: 0,
        igst_amount: "",
        total_amount: ""
      }
    ],
    valid: true,
    RequiredRule: [v => !!v || "This is required"],
    IntegerNumberRule: [
      v => !!v || "This is required",
      v => Number.isInteger(Number(v)) || "The value must be a integer"
    ]
  }),

  methods: {
    saveInvoice() {
      console.log(JSON.stringify(this.productList));
    },
    calculateTotal() {
      this.TotalAmountWithoutTax = this.productList.reduce(function(sum, product) {
        var lineAmountWithoutTax = parseFloat(product.amount);
        if (!isNaN(lineAmountWithoutTax)) {
          return sum + lineAmountWithoutTax;
        }
      }, 0);
      this.TotalAmountWithoutTax = this.TotalAmountWithoutTax.toFixed(2);


this.CGSTTotal = this.productList.reduce(function(sum, product) {
        var CGSTTotal = parseFloat(product.cgst_amount);
        if (!isNaN(CGSTTotal)) {
          return sum + CGSTTotal;
        }
      }, 0);
this.CGSTTotal = this.CGSTTotal.toFixed(2);




this.SGSTTotal = this.productList.reduce(function(sum, product) {
        var SGSTTotal = parseFloat(product.sgst_amount);
        if (!isNaN(SGSTTotal)) {
          return sum + SGSTTotal;
        }
      }, 0);
this.SGSTTotal = this.SGSTTotal.toFixed(2);




this.IGSTTotal = this.productList.reduce(function(sum, product) {
        var IGSTTotal = parseFloat(product.igst_amount);
        if (!isNaN(IGSTTotal)) {
          return sum + IGSTTotal;
        }
      }, 0);
this.IGSTTotal = this.IGSTTotal.toFixed(2);



this.TotalAmountWithTax = this.productList.reduce(function(sum, product) {
        var TotalAmountWithTax = parseFloat(product.total_amount);
        if (!isNaN(TotalAmountWithTax)) {
          return sum + TotalAmountWithTax;
        }
      }, 0);
this.TotalAmountWithTax = this.TotalAmountWithTax.toFixed(2);

 this.AmountinWords=this.convertToWords(this.TotalAmountWithTax);


    //   total = subtotal * (18 / 100) + subtotal;
    //   total = parseFloat(total);
    //   if (!isNaN(total)) {
    //     this.invoice_total = total.toFixed(2);
    //   } else {
    //     this.invoice_total = "0.00";
    //   }
    },
    calculateLineTotal(productList) {
      var LineTotalWithoutTax =
        parseFloat(productList.rate) * parseFloat(productList.quantity);
      var tax1_amount =
        (parseFloat(LineTotalWithoutTax) * parseFloat(productList.cgst_rate)) /
        100;
      var tax2_amount =
        (parseFloat(LineTotalWithoutTax) * parseFloat(productList.sgst_rate)) /
        100;
      var tax3_amount =
        (parseFloat(LineTotalWithoutTax) * parseFloat(productList.igst_rate)) /
        100;
      if (!isNaN(LineTotalWithoutTax)) {
        productList.amount = LineTotalWithoutTax;
        productList.cgst_amount = tax1_amount;
        productList.sgst_amount = tax2_amount;
        productList.igst_amount = tax3_amount;
        var LineTotalWithTax =
          LineTotalWithoutTax + tax1_amount + tax2_amount + tax3_amount;
        productList.total_amount = Math.round(LineTotalWithTax);
      }
      this.calculateTotal();
    },
    deleteRow(index, productList) {
      var idx = this.productList.indexOf(productList);
      console.log(idx, index);
      if (idx > -1) {
        this.productList.splice(idx, 1);
      }
      this.calculateTotal();
    },
    addNewRow() {
      this.productList.push({
        description: "",
        hsn_sac: "",
        quantity: "",
        unit: "",
        rate: "",
        amount: "",
        cgst_rate: 0,
        cgst_amount: "",
        sgst_rate: 0,
        sgst_amount: "",
        igst_rate: 0,
        igst_amount: "",
        total_amount: ""
      });
    },


convertToWords(total_withtax)
{
//convert Amount to Words
var amount=total_withtax;
var words = new Array();
words[0] = '';
words[1] = 'One';
words[2] = 'Two';
words[3] = 'Three';
words[4] = 'Four';
words[5] = 'Five';
words[6] = 'Six';
words[7] = 'Seven';
words[8] = 'Eight';
words[9] = 'Nine';
words[10] = 'Ten';
words[11] = 'Eleven';
words[12] = 'Twelve';
words[13] = 'Thirteen';
words[14] = 'Fourteen';
words[15] = 'Fifteen';
words[16] = 'Sixteen';
words[17] = 'Seventeen';
words[18] = 'Eighteen';
words[19] = 'Nineteen';
words[20] = 'Twenty';
words[30] = 'Thirty';
words[40] = 'Forty';
words[50] = 'Fifty';
words[60] = 'Sixty';
words[70] = 'Seventy';
words[80] = 'Eighty';
words[90] = 'Ninety';
amount = amount.toString();
var atemp = amount.split(".");
var number = atemp[0].split(",").join("");
var n_length = number.length;
var words_string = "";
if (n_length <= 9) {
  var n_array = new Array(0, 0, 0, 0, 0, 0, 0, 0, 0);
  var received_n_array = new Array();
  for (var i = 0; i < n_length; i++) {
      received_n_array[i] = number.substr(i, 1);
  }
  var j=0;
  for (i = 9 - n_length, j = 0; i < 9; i++, j++) {
      n_array[i] = received_n_array[j];
  }
  for ( i = 0, j = 1; i < 9; i++, j++) {
      if (i == 0 || i == 2 || i == 4 || i == 7) {
          if (n_array[i] == 1) {
              n_array[j] = 10 + parseInt(n_array[j]);
              n_array[i] = 0;
          }
      }
  }
  var value = "";
  for ( i = 0; i < 9; i++) {
      if (i == 0 || i == 2 || i == 4 || i == 7) {
          value = n_array[i] * 10;
      } else {
          value = n_array[i];
      }
      if (value != 0) {
          words_string += words[value] + " ";
      }
      if ((i == 1 && value != 0) || (i == 0 && value != 0 && n_array[i + 1] == 0)) {
          words_string += "Crores ";
      }
      if ((i == 3 && value != 0) || (i == 2 && value != 0 && n_array[i + 1] == 0)) {
          words_string += "Lakhs ";
      }
      if ((i == 5 && value != 0) || (i == 4 && value != 0 && n_array[i + 1] == 0)) {
          words_string += "Thousand ";
      }
      if (i == 6 && value != 0 && (n_array[i + 1] != 0 && n_array[i + 2] != 0)) {
          words_string += "Hundred ";
      } else if (i == 6 && value != 0) {
          words_string += "Hundred ";
      }
  }
  words_string+= "Only";
  words_string = words_string.split("  ").join(" ");
  }

return words_string;

},

    isNumber: function(evt) {
      evt = evt ? evt : window.event;
      var charCode = evt.which ? evt.which : evt.keyCode;
      if (
        charCode > 31 &&
        (charCode < 48 || charCode > 57) &&
        charCode !== 46
      ) {
        evt.preventDefault();
      } else {
        return true;
      }
    }
  }
};
</script>
