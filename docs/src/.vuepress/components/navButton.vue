<template>
    <div class="home-buttons">
      <router-link to="/devguide/" class="home-button">Developer Guide</router-link>
      <router-link to="/user/" class="home-button">User Guide</router-link>
    </div>
  </template>
  
  <style scoped>
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