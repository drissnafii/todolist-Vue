<script setup lang="ts"></script>

<template>
  <div class="app-container">
    <nav>
      <router-link to="/">Home</router-link>
      <router-link to="/about">About</router-link>
      <router-link to="/mytodo">My Todo</router-link>
    </nav>
    <router-view />
  </div>
</template>

<style>
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

.app-container {
  background: linear-gradient(
    135deg,
    #fff8e1 0%,
    /* Premium warm cream */ #f5f5f5 50%,
    /* Light gray middle */ #e6e6e6 100% /* Subtle gray depth */
  );
  min-height: 100vh;
  margin: 0;
  padding: 0;
  color: #333;
  background-attachment: fixed;
}
</style>
