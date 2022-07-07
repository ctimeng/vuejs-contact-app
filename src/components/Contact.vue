<template>
  <div class="container">
    <div class="card mx-auto mb-3 mt-3 shadow" style="width: 30rem">
      <div class="card-body">
        <ValidationObserver v-slot="{ invalid }" ref="observer">
          <form @submit.prevent="onSubmit">
            <ValidationProvider
              name="Name"
              rules="required"
              v-slot="{ errors }"
            >
              <div class="mb-3">
                <input
                  type="text"
                  class="form-control"
                  placeholder="Name"
                  v-model="contact.name"
                />
              </div>
              <span>{{ errors[0] }}</span>
            </ValidationProvider>
            <ValidationProvider
              name="Phone"
              :rules="{
                required: true,
                regex: /^\+?[0-9 ]+$/,
              }"
              v-slot="{ errors }"
            >
              <div class="mb-3">
                <input
                  type="text"
                  class="form-control"
                  placeholder="Phone"
                  v-model="contact.phone"
                />
              </div>
              <span>{{ errors[0] }}</span>
            </ValidationProvider>
            <ValidationProvider
              name="E-mail"
              rules="required|email"
              v-slot="{ errors }"
            >
              <div class="mb-3">
                <input
                  type="text"
                  class="form-control"
                  placeholder="john@example.com"
                  v-model="contact.email"
                />
              </div>
              <span>{{ errors[0] }}</span>
            </ValidationProvider>
            <div class="mb-3 d-grid gap-2">
              <button
                type="submit"
                id="btnSave"
                class="btn btn-success"
                :disabled="invalid"
              >
                {{ saveUpdateCaption }}
              </button>
            </div>
          </form>
        </ValidationObserver>
      </div>
    </div>

    <table class="table">
      <thead>
        <tr>
          <th scope="col">№</th>
          <th scope="col">Name</th>
          <th scope="col">Phone</th>
          <th scope="col">Email</th>
          <th scope="col">Date</th>
          <th scope="col" style="width: 10rem"></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(contact, i) in contacts" :key="contact.id">
          <td>{{ i + 1 }}</td>
          <td>{{ contact.name }}</td>
          <td>{{ contact.phone }}</td>
          <td>{{ contact.email }}</td>
          <td>{{ moment(contact.date,'YYYY-MM-DD HH:mm:ss').format("DD-MM-YYYY HH:mm:ss") }}</td>
          <td>
            <a href="#" class="btn btn-primary" @click.prevent="onEdit(i)"
              >Edit</a
            >
            <a href="#" class="btn btn-danger ms-1" @click.prevent="onDelete(i)"
              >Delete</a
            >
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import { required, digits, email, max, regex } from "vee-validate/dist/rules";
import {
  extend,
  ValidationObserver,
  ValidationProvider,
  setInteractionMode,
} from "vee-validate";
import moment from 'moment';

setInteractionMode("eager");

extend("digits", {
  ...digits,
  message: "{_field_} needs to be {length} digits. ({_value_})",
});

extend("required", {
  ...required,
  message: "{_field_} can not be empty",
});

extend("max", {
  ...max,
  message: "{_field_} may not be greater than {length} characters",
});

extend("regex", {
  ...regex,
  message: "{_field_} {_value_} does not match {regex}",
});

extend("email", {
  ...email,
  message: "Email must be valid",
});

const EMPTY_VALUE = -1

export default {
  name: "Contact",
  components: {
    ValidationObserver,
    ValidationProvider
  },
  data: () => ({
    contact: {
        id: 0,
        name: null,
        phone: null,
        email: null,
        date:""
    },
    contactCopy: {},
    contacts: [],
    selectedIndex: -1,
    EMPTY_VALUE
  }),
  computed: {
    autoIncrementContactId() {
      return this.contacts.length > 0
        ? this.contacts[this.contacts.length - 1].id + 1
        : 1;
    },

    isSave(){
        return this.selectedIndex == EMPTY_VALUE
    },

    saveUpdateCaption(){
        return this.isSave ? 'Save': 'Update'
    }
  },
  created() {
    this.contacts = JSON.parse(localStorage.getItem("contacts") || "[]");
    this.moment = moment;
  },
  mounted() {
        this.contactCopy = { ...this.contact };
    },
  methods: {
    onSubmit() {
        const self = this
      var exists = false;
      this.contacts.forEach(function (contact) {
        if (
          contact.id != self.contact.id &&
          (contact.phone == self.contact.phone || contact.email == self.contact.email)
        ) {
          exists = true;
        }
      });

      if (exists) {
        alert("Record already exists!");
        return;
      }
      if (self.isSave) {
        this.contact.id = this.autoIncrementContactId
        this.contact.date = new Date()
        this.contacts.push(this.contact);
      } else {
        this.contacts[this.selectedIndex] = this.contact;
      }

      localStorage.setItem("contacts", JSON.stringify(this.contacts));
      this.clear();
      alert(this.saveUpdateCaption+' successfully!')
    },

    onEdit(i) {
      this.selectedIndex = i;
      this.contact = this.contacts[i];
    },

    onDelete(i) {
      let contact = this.contacts[i];
      if (
        window.confirm(
          "Are you sure, you want to delete [" + contact.name + "] !"
        )
      ) {
        this.contacts.splice(i, 1);
        localStorage.setItem("contacts", JSON.stringify(this.contacts));
      }
    },

    clear() {
      this.selectedIndex = EMPTY_VALUE;
      this.resetFormFields()
      this.$nextTick(() => this.$refs.observer.reset());
    },

    resetFormFields() {
      this.contact = { ...this.contactCopy };
    },
    moment: function () {
        return moment();
    }
  },
};
</script>

<style scoped>
span {
  display: block;
  color:red
}
</style>
