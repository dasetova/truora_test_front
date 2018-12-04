<template>
  <div id="app">
    <div>
      <b-card title="Recipes Module">
        
        <b-form @submit="onSubmit" @reset="onReset" v-if="show">
          <b-form-group id="exampleInputGroup2"
                        label="Recipe Name:"
                        label-for="exampleInput2">
            <b-form-input id="exampleInput2"
                          type="text"
                          v-model="form.name"
                          required
                          placeholder="Enter name">
            </b-form-input>
          </b-form-group>
          <b-form-group id="exampleInputGroup2"
                        label="Description:"
                        label-for="exampleInput2">
            <b-form-input id="exampleInput2"
                          type="text"
                          v-model="form.description"
                          required
                          placeholder="Enter recipe description">
            </b-form-input>
          </b-form-group>
          <b-form-group id="exampleInputGroup2"
                        label="Indications:"
                        label-for="areatxt_indications">
            <b-form-textarea id="areatxt_indications"
                          v-model="form.indications"
                          required
                          placeholder="Enter recipe indications"
                          :rows="3"
                          :max-rows="6"
                          >
            </b-form-textarea>
          </b-form-group>
          <b-form-group id="exampleInputGroup3"
                        label="Category:"
                        label-for="exampleInput3">
            <b-form-select id="exampleInput3"
                          :options="categories"
                          required
                          v-model="form.category_id">
            </b-form-select>
          </b-form-group>
          <b-form-group id="exampleInputGroup2"
                        label="Ingredients:"
                        label-for="Ingredients">
          <b-input-group prepend="Ingredients">
            <b-form-input v-model="form.ingredient_description"
                  type="text"
                  placeholder="Enter ingredient"></b-form-input>
            <b-form-input v-model="form.ingredient_quantity"
                  type="number"
                  placeholder="Quantity"></b-form-input>
            <b-form-select id="ingredient_measure"
                          :options="form.measures"
                          required
                          v-model="form.ingredient_measure_unit">
            </b-form-select>
            
            <b-btn variant="info" v-on:click="addIngredient">Add ingredient</b-btn>
            
          </b-input-group>
          </b-form-group>
          <b-button type="submit" variant="primary">Submit</b-button>
          <b-button type="reset" variant="danger">Reset</b-button>
        </b-form>
      </b-card>
    </div>
    <b-table show-empty 
            responsive
            :items="recipes" :fields="fields">
      <template slot="category" slot-scope="data">
        {{data.name}}
      </template>        
    </b-table>
    
  </div>
</template>

<script>

import axios from 'axios'
export default {
  name: 'app',
  data () {
    return {
      form: {
        name: '',
        description: '',
        category_id: null,
        indications: '',
        measures: ['gr', 'ml', 'cup', 'units'],
        ingredient_description: '',
        ingredient_quantity: 0,
        ingredient_measure_unit: '' 
      },
      categories: [],
      show: true,
      fields: {
        id: {
          sortable: true
        },
        name: {
          label: 'Name',
          sortable: true
        },
        description: {
          label: 'Description'
        },
        indications: {
          key: 'indications'
        },
        category: {
          key: 'category',
          label: 'Recipe Category'
        },
        ingredients: {
          key: 'ingredients',
          label: 'Ingredients'
        }
      },
      recipes: [],
      errors: [],
      ingredients: []
    }
  }, 
  methods: {
    addIngredient (evt) {
      this.ingredients.push({
        description: this.form.ingredient_description,
        quantity: parseInt(this.form.ingredient_quantity, 10),
        measure_unit: this.form.ingredient_measure_unit,
      })
      this.form.ingredient_description=''
      this.form.ingredient_quantity=0
      this.form.ingredient_measure_unit=''
    },
    onSubmit (evt) {
      alert(JSON.stringify(this.form));
      axios.post('http://localhost:6767/recipes', {
        name: this.form.name,
        description: this.form.description,
        category_id: this.form.category_id,
        indications: this.form.indications,
        ingredients: this.ingredients,
      })
      .then(function (response) {
        console.log(response);
        alert("Recipe created")
      })
      .catch(function (error) {
        alert("Recipe can't be created")
        console.log(error);
      });
      evt.preventDefault();
    },
    onReset (evt) {
      evt.preventDefault();
      /* Reset our form values */
      this.form.name = '';
      this.form.description = '';
      this.form.category = null;
      this.form.indications = '';
      /* Trick to reset/clear native browser form validation state */
      this.show = false;
      this.recipeCreatedAlert = false;
      this.recipeNoCreatedAlert = false;
      this.$nextTick(() => { this.show = true });
    }
  },

  created() {
    axios.get(`http://localhost:6767/recipes`)
    .then(response => {
      this.recipes = response.data
    })
    .catch(e => {
      this.errors.push(e)
    })

    axios.get(`http://localhost:6767/categories`)
    .then(response => {
      var options = []
      console.log("data")
      console.log(response.data)
      console.log("Length:" + response.data.length)
      
      for (var i = 0; i < response.data.length; i++) {
          options.push({ value: response.data[i].id, text: response.data[i].name })
      } 
      this.categories = options
    })
    .catch(e => {
      this.errors.push(e)
    })
  }
}
</script>
