<template>
<div class="home">
  <div>
    <h2>Post something!</h2>
  </div>
  <div class="add">
    <div class="form">
      <input v-model="poster" placeholder="Who are you?">
      <p></p>
      <textarea v-model="post" placeholder="What's on your mind?"></textarea>
      <p></p>
      <input type="file" name="photo" @change="fileChanged">
      <button @click="upload">Post!</button>
    </div>
    <div class="image" v-if="addItem">
      <h2>{{addItem.poster}}       {{addItem.timestamp}}</h2>
      <h3>{{addItem.post}}</h3>
      <img :src="addItem.path"/>
    </div>
  </div>
  <br/>
  <section class="image-gallery">
    <div class="image" v-for="item in items" :key="item.id">
      <h2>{{item.poster}}&emsp;{{item.timestamp}}</h2>
      <h3>{{item.post}}</h3>
      <img :src="item.path" />
    </div>
  </section>
</div>
</template>

<style scoped>
.image h2 {
  font-style: italic;
}

.add {
  display: flex;
}

input,
textarea,
button {
  font-family: "Montserrat", sans-serif;
  font-size: 1em;
}

.form {
  margin-right: 50px;
}

.upload h2 {
  margin: 0px;
}

upload img {
  max-width: 300px;
}

/* Masonry */
*,
*:before,
*:after {
  box-sizing: inherit;
}

.image-gallery {
  column-gap: 1.5em;
}

.image {
  margin: 0 0 1.5em;
  display: inline-block;
  width: 100%;
  border-style: solid;
  border-radius: 10px;
  border-color: #657786;
}

.image h2 {
  margin-left: 3px;
}

.image h3 {
  margin-left: 3px;
}


.image img {
  width: 90%;
  margin-left: 3px;
  margin-bottom: 3px;
}

/* Masonry on large screens */
@media only screen and (min-width: 1024px) {
  .image-gallery {
    column-count: 4;
  }
}

/* Masonry on medium-sized screens */
@media only screen and (max-width: 1023px) and (min-width: 768px) {
  .image-gallery {
    column-count: 3;
  }
}

/* Masonry on small screens */
@media only screen and (max-width: 767px) and (min-width: 540px) {
  .image-gallery {
    column-count: 2;
  }
}
</style>

<script>
// @ is an alias to /src
import axios from 'axios';
export default {
  name: 'Home',
  data() {
    return {
      poster: "",
      post: "",
      file: null,
      addItem: null,
      items: [],
    }
  },
  created() {
    this.getItems();
  },
  methods: {
    async getItems() {
      try {
        let response = await axios.get("/api/posts");
	this.items = response.data;
	return true;
      } catch (error) {
        console.log(error);
      }
    },
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
	timestamp: this.getTimestamp(),
	path: r1.data.path
      });
      this.addItem = r2.data;
     }
     catch (error) {
      console.log(error);
     }
    },
    getTimestamp() {
      let today = new Date();
      let timestamp = (today.getMonth()+1) + "/" + today.getDate() + "/" + today.getFullYear() + "  " + today.getHours().toLocaleString('en-US', {minimumIntegerDigits: 2, useGrouping:false}) + ":" + today.getMinutes().toLocaleString('en-US', {minimumIntegerDigits: 2, useGrouping:false});
      return timestamp;
    }
  }
}
</script>
