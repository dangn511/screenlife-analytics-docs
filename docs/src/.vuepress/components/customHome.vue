<template>
    <div class="home-page">
      <div class="hero">
        <!-- <img :src="heroImage" alt="Hero Image"> -->
        <h1>{{ heroTitle }}</h1>
        <p>{{ description }}</p>
      </div>
      <hr>
      <div class="home-buttons">
        <router-link to="/devguide/" class="home-button">Developer Guide</router-link>
        <router-link to="/user/" class="home-button">User Guide</router-link>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: {
    //   heroImage: String,
      heroTitle: String,
      description: String
    }
  }
  </script>
  
  <style scoped>
  .hero {
    text-align: center;
  }
  
  .home-page img {
    max-width: 100%;
  }
  
  .home-buttons {
    display: flex;
    justify-content: space-around;
    margin-top: 20px;
  }
  
  .home-button {
    display: inline-block;
    padding: 10px 20px;
    background: #3eaf7c;
    color: white;
    text-decoration: none;
    border-radius: 4px;
    font-size: 18px;
  }
  </style>