<script setup lang="ts"></script>

<template>
  <nav>
    <router-link to="/">Home</router-link>
    <router-link to="/about">About</router-link>
    <router-link to="/mytodo">My Todo</router-link>
  </nav>
  <router-view />
</template>

<style scoped>
nav {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 30px;
  padding: 20px;
  background-color: #f8f9fa;
  border-bottom: 2px solid #e0e0e0;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

nav a {
  text-decoration: none;
  color: #333;
  font-weight: 600;
  font-size: 16px;
  padding: 8px 16px;
  border-radius: 4px;
  transition: all 0.3s ease;
}

nav a:hover {
  background-color: #e9ecef;
  color: #42b983;
}

nav a.router-link-active {
  color: #42b983;
  background-color: #e9f7f0;
}
</style>
