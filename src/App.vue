<template>
  <div class="container">
    <div class="row">
      <div class="col">
        <h2>List Form</h2>
        <table class="table">
          <thead>
            <tr>
              <th scope="col">Name</th>
              <th scope="col">Description</th>
              <th scope="col">Category</th>
              <th scope="col">Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="f in forms" :key="f.id">
              <td>{{ f.name }}</td>
              <td>{{ f.description }}</td>
              <td>{{ f.category ? f.category.name : 'not found' }}</td>
              <td>
                <button class="btn btn-warning" @click="editForm(f)">Edit</button>
                <button class="btn btn-danger" @click="deleteForm(f.id)">Delete</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="col">
        <h2>{{ editing ? 'Update Form' : 'Add Form'}}</h2>
        <form @submit.prevent="saveForm">
          <input type="text" v-model="form.name" placeholder="Name">
          <input type="text" v-model="form.description" placeholder="Description">
          <select v-model="form.category_id" required>
            <option v-for="c in categories" :key="c.id" :value="c.id">{{ c.name }}</option>
          </select>
          <button class="btn btn-primary">{{ editing ? 'Update' : 'Add' }}</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>

import { ref, onMounted } from "vue";

const forms = ref([]);
const categories = ref([]);
const form = ref({ id: null, category_id: "", name: "", description: "" });
const editing = ref(false);

onMounted(async () => {
  const FormRes = await fetch('http://127.0.0.1:8000/forms')
  forms.value = await FormRes.json();

  const CategoryRes = await fetch('http://127.0.0.1:8000/categories')
  categories.value = await CategoryRes.json();
});

const saveForm = async () => {
  if (editing.value) {
    await fetch(`http://127.0.0.1:8000/forms/${form.value.id}`, {
      method : "PUT",
      headers: {"Content-type": "application/json"},
      body: JSON.stringify(form.value),
    });
    editing.value = false;
  } else {
    await fetch("http://127.0.0.1:8000/forms", {
      method : "POST",
      headers: {"Content-type": "application/json"},
      body: JSON.stringify(form.value),
    });
  }

  form.value = { id: null, category_id: "", name: "", description: "" }
  const res = await fetch('http://127.0.0.1:8000/forms');
  forms.value = await res.json();
};

const editForm = (f) => {
  form.value = { ...f };
  editing.value = true;
};

const deleteForm = async (id) => {
  await fetch(`http://127.0.0.1:8000/forms/${id}`, {
    method : "DELETE",
  });

  const res = await fetch('http://127.0.0.1:8000/forms');
  forms.value = await res.json();
};

</script>
