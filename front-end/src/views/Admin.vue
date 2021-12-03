<template>
<div class="admin">
  <h1>The Back Room</h1>
  <h2>For when people post what they probably shouldn't</h2>
    <div class="heading">
      <h2>Edit/Delete an Item by User</h2>
    </div>
    <div class="edit">
      <div class="form">
        <input v-model="findTitle" placeholder="Search Users">
        <div class="suggestions" v-if="suggestions.length > 0">
          <div class="suggestion" v-for="s in suggestions" :key="s.id" @click="selectItem(s)">{{s.poster}}
          </div>
        </div>
      </div>
      <div class="found" v-if="findItem">
        <input v-model="findItem.poster">
	<p>{{findItem.timestamp}}</p>
	<p></p>
	<textarea v-model="findItem.post"></textarea>
        <p></p>
        <img :src="findItem.path" />
      </div>
      <div class="actions" v-if="findItem">
        <button @click="deleteItem(findItem)">Delete</button>
	<button @click="editItem(findItem)">Edit</button>
      </div>
    </div>
</div>
</template>

<style scoped>
.image h2 {
  font-style: italic;
  font-size: 1em;
}

.heading {
  display: flex;
  margin-bottom: 20px;
  margin-top: 20px;
}

.heading h2 {
  margin-top: 8px;
  margin-left: 10px;
}

.edit {
  display: flex;
}

/* Form */
input,
textarea,
select,
button {
  font-family: 'Montserrat', sans-serif;
  font-size: 1em;
}

.form {
  margin-right: 50px;
}

/* Found Posts */
.found {
  margin: 0 0 1.5em;
  display: inline-block;
  width: 100%;
  border-style: solid;
  border-radius: 10px;
  border-color: #657786;
}

.found p {
  margin-left: 3px;
}

.found input {
  margin-top: 3px;
  margin-left: 3px;
}

.found textarea {
  margin-left: 3px;
}

.found img {
  max-width: 300px;
  margin-left: 3px;
  margin-bottom: 3px;
}

/* Suggestions */
.suggestions {
  width: 200px;
  border: 1px solid #ccc;
}

.suggestion {
  min-height: 20px;
}

.suggestion:hover {
  background-color: #5BDEFF;
  color: #fff;
}
</style>

<script>
import axios from 'axios';
export default {
  name: 'Admin',
  data() {
    return {
      poster: "",
      post: "",
      file: null,
      addItem: null,
      items: [],
      findTitle: "",
      findItem: null,
    }
  },
  computed: {
    suggestions() {
      let items = this.items.filter(item => item.poster.toLowerCase().startsWith(this.findTitle.toLowerCase()));
      return items.sort((a, b) => a.title > b.title);
    }
  },
  created() {
    this.getItems();
  },
  methods: {
    fileChanged(event) {
      this.file = event.target.files[0]
    },
    async upload() {
      try {
        const formData = new FormData();
	formData.append('photo', this.file, this.file.name)
	let r1 = await axios.post('/api/pictures', formData);
	let r2 = await axios.post('/api/posts', {
          poster: this.poster,
          post: this.post,
          timestam: this.getTimestamp(),
          path: r1.data.path
	});
	this.addItem = r2.data;
      } catch (error) {
        console.log(error);
      }
    },
    async getItems() {
      try {
	let response = await axios.get("/api/posts");
	this.items = response.data;
	return true;
      } catch(error) {
        console.log(error);
      }
    },
    selectItem(item) {
      this.findTitle = "";
      this.findItem = item;
    },
    async deleteItem(item) {
      try {
        await axios.delete("/api/posts/" + item._id);
	this.findItem = null;
	this.getItems();
	return true;
      } catch (error) {
        console.log(error);
      }
    },
    async editItem(item) {
      try {
        await axios.put("/api/posts/" + item._id, {
          poster: this.findItem.poster,
          post: this.findItem.post,
	});
	this.findItem = null;
	this.getItems();
	return true;
      } catch (error) {
        console.log(error);
      }
    },
    getTimestamp() {
      let today = new Date();
      let timestamp = (today.getMonth()+1) + "/" + today.getDate() + "/" + today.getFullYear() + "  " + today.getHours() + ":" + today.getMinutes();
      return timestamp;
    },
  }
}
</script>
