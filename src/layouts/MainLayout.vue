<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-toolbar-title>Movie Library</q-toolbar-title>
      </q-toolbar>
    </q-header>
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
import { provide, ref } from 'vue'

export interface Movies {
  id: number | null;
  title: string;
  director: string;
  summary: string;
  genres: string[];
}

const movies = ref<Movies[]>([
  {
    id: 1,
    title: 'Sharknado',
    director: 'Anthony C. Ferrante',
    summary: 'Off the coast of Mexico, the ship of shark fin smuggler Carlos Santiago is caught in a hurricane.',
    genres: ['Action', 'Sci-fi']
  },
  {
    id: 2,
    title: 'Sharknado 2: The Second One',
    director: 'Anthony C. Ferrante',
    summary: 'Fin Shepard, and his former wife, April Wexler, are traveling to New York City.',
    genres: ['Drama', 'Sci-fi']
  },
  {
    id: 3,
    title: 'Sharknado 3: Oh Hell No!',
    director: 'Anthony C. Ferrante',
    summary: 'In Washington, D.C., Fin Shepard (Ian Ziering) attends an awards ceremony at the White House.',
    genres: ['Animation', 'Sci-fi']
  }
])

provide('movies', movies)
</script>
