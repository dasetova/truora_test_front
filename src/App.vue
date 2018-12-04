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
                          v-model="form.ingredient_measure_unit">
            </b-form-select>
            
            <b-btn variant="info" v-on:click="addIngredient">Add ingredient</b-btn>
            
          </b-input-group>
          </b-form-group>
          <b-form-group id="exampleInputGroup2" >
              <b-form-input v-model="form.added_ingredients"
                  type="text" plaintext></b-form-input>
          </b-form-group>
          
          <b-button type="submit" variant="primary">Submit</b-button>
          <b-button type="reset" variant="danger">Reset</b-button>
        </b-form>
      </b-card>
    </div>
    <b-table show-empty 
            striped hover
            :items="recipes" :fields="fields">
       <template slot="actions" slot-scope="data">
          <b-btn variant="info" v-on:click="addIngredient">Edit</b-btn>
          <b-btn variant="danger" v-on:click="onDelete(data.item.id)">Delete</b-btn>
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
        ingredient_measure_unit: '',
        added_ingredients: '', 
      },
      categories: [],
      show: true,
      fields: [
        {
          key: 'id',
          sortable: true
        },
        {
          key: 'name',
          sortable: true
        },
        'indications',
        {
          key: 'category',
          formatter: 'category_name'
        },
        {
          key: 'ingredients',
          formatter: 'ingredients_to_string'
        },
        'actions',
      ],
      recipes: [],
      errors: [],
      ingredients: []
    }
  }, 
  methods: {
    category_name(value){
      return `${value.name.charAt(0).toUpperCase() + value.name.slice(1)}`
    },
    ingredients_to_string(value){
      if (value != null) {
        var ingredients_formatted = ""
        for (var i = 0; i < value.length; i++) {
            ingredients_formatted += value[i].description + " - " + value[i].quantity + " " + value[i].measure_unit + ",\n" 
        }
        return `${ingredients_formatted}`
      }
      return `${"Recipe has no ingredients registered"}`
    },
    addIngredient (evt) {
      this.ingredients.push({
        description: this.form.ingredient_description,
        quantity: parseInt(this.form.ingredient_quantity, 10),
        measure_unit: this.form.ingredient_measure_unit,
      })
      this.form.added_ingredients += 
        this.form.ingredient_description + " - " + this.form.ingredient_quantity + " " + this.form.ingredient_measure_unit + "," 
      this.clearIngredientForm()
    },
    onSubmit (evt) {
      axios.post('http://localhost:6767/recipes', {
        name: this.form.name,
        description: this.form.description,
        category_id: this.form.category_id,
        indications: this.form.indications,
        ingredients: this.ingredients,
      })
      .then(function (response) {
        alert("Recipe created");
      })
      .catch(function (error) {
        console.log(error)
        alert("Recipe can't be created")
      });
      this.get_recipes();
      evt.preventDefault();
    },
    onReset (evt) {
      evt.preventDefault();
      this.clearForm();
      this.$nextTick(() => { this.show = true });
    },
    onDelete (recipe_id) {
      axios.delete('http://localhost:6767/recipes/' + recipe_id)
      .then(function (response) {
        alert("Recipe deleted");
      })
      .catch(function (error) {
        console.log(error)
        alert("Recipe can't be deleted")
      });
      this.get_recipes();
    },
    clearForm(){
      this.form.name = '';
      this.form.description = '';
      this.form.category = null;
      this.form.indications = '';
      this.show = false;
      this.recipeCreatedAlert = false;
      this.recipeNoCreatedAlert = false;
      this.form.added_ingredients='';
      this.clearIngredientForm()  
    },
    clearIngredientForm(){
      this.form.ingredient_description=''
      this.form.ingredient_quantity=0
      this.form.ingredient_measure_unit=''
    },
    get_recipes(){
      axios.get(`http://localhost:6767/recipes`)
      .then(response => {
        this.recipes = response.data
      })
      .catch(e => {
        console.log(e)
      })
    }
  },
  created() {
    this.get_recipes()  

    axios.get(`http://localhost:6767/categories`)
    .then(response => {
      var options = []
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
