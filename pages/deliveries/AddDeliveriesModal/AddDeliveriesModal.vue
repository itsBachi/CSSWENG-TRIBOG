<template>
  <b-modal
    id="add-delivery-modal"
    centered
    title="Add Delivery"
    modal-class="generic-modal"
  >
    <b-form class="generic-form">
      <b-form-group>
        <b-row>
          <b-col>
            <!-- product select -->
            <CommonProductSelect
              type="product"
              refName="productSelect"
            />
          </b-col>
          <b-col>
            <!-- status -->
            <CommonStatusSelect
              type="status"
              v-on:statusChanges="$event => setStatus($event)"
            />
            
          </b-col>
        </b-row>
      </b-form-group>
      <b-form-group>
        <b-row>
          <b-col>
            <!-- expected quantity -->
            <label for="expected-quantity">Expected Quantity <span>*</span></label>
            <b-form-input 
              id="expected-quantity"
              type="number"
              min="0"
              placeholder="Enter Expected Quantity"
              required
              v-model="formData.expected_quantity"
              @keydown="preventDecimal"
              @blur="setExpQuantityToZero"
            ></b-form-input>
          </b-col>
          <b-col>
            <!-- current quantity -->
            <label for="current-quantity">Current Quantity <span>*</span></label>
            <b-form-input 
              id="current-quantity"
              type="number"
              min="0"
              placeholder="Enter Current Quantity"
              required
              v-model="formData.current_quantity"
              @keydown="preventDecimal"
              @blur="setCurrQuantityToZero"
            ></b-form-input>
          </b-col>
        </b-row>
      </b-form-group>
    </b-form>

    <template #modal-footer="{cancel}">
      <b-button @click="cancel()">Cancel</b-button>
      <b-button
        type="submit"
        class="btn-info"
        @click="submit"
        :disabled="submitting || !enableSubmit"
        >Add</b-button
      >
    </template>
  </b-modal>
</template>

<style>
</style>

<script>
import Helpers from '~/mixins/Helpers'

export default {
  mixins: [Helpers],

  data() {
    return {
      submitting: false,

      formData: {
        product_id: 0,
        expected_quantity: 0,
        status: '',
        current_quantity: 0,
      }
    }
  },

  methods: {
    async submit(){
      this.submitting = true
      
      try{
        await this.$axios.$post(`${process.env.baseUrl}/deliveries-create/`, this.requestParams)
      } catch(e) {
        console.log(e)
      } finally {
        this.submitting = false

        this.$bvModal.hide('add-delivery-modal')

        this.$nuxt.$emit('addedDelivery', true)

        this.resetFormData()
      }
    },

    setExpQuantityToZero() {
      if(!this.formData.expected_quantity) {
        this.formData.expected_quantity = 0;
      }
    },

    setProduct(product) {
      this.formData.product_id = product?.id ;
    },

    setStatus(status) {
      this.formData.status = status ? status.name : '';
    },
    
    setCurrQuantityToZero(){
      if(!this.formData.current_quantity) {
        this.formData.current_quantity = 0;
      }
    },

    resetFormData() {
      this.formData = {
        product_id: null,
        expected_quantity: 0,
        status: '',
        current_quantity: 0,
      }
    }
  },

  computed: {
    enableSubmit() {
      return (
        this.productIdParams &&
        this.formData.status &&
        this.expQuantityParams &&
        this.currQuantityParams
      )
    },

    productIdParams() {
      return(this.formData.product_id >= 0 && this.formData.product_id !== null)
    },

    expQuantityParams() {
      return(this.formData.expected_quantity >= 0)
    },

    currQuantityParams() {
      return(this.formData.current_quantity >= 0)
    },

    requestParams() {
      return {
        ...this.formData,
        product_id: Number(this.formData.product_id),
        expected_quantity: Number(this.formData.expected_quantity),
        status: this.formData.status,
        current_quantity: Number(this.formData.current_quantity),
      }
    },
  },

  created() {
    this.$nuxt.$on('productChanges-productSelect', (product_id) => {
      this.setProduct(product_id)
    })
  }
}
</script>