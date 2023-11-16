<script setup lang="ts">
import { inject, reactive, ref, watch } from 'vue'
import type { Ref } from 'vue'
import type { Movies } from '../layouts/MainLayout.vue'

const movies = inject<Ref<Movies[]>>('movies')
const displayMovies = ref(movies?.value)
const searchQuery = ref<string>('')
const formDialog = ref<boolean>(false)
const formData = reactive<Movies>({
  id: null,
  title: '',
  director: '',
  summary: '',
  genres: []
})
const genres = ['Action', 'Animation', 'Drama', 'Sci-fi']
const formMessage = reactive({
  genres: ''
})

/**
 * Watcher to update list of movies to be displayed after search input are being filled.
 */
watch(searchQuery, (q) => {
  if (q) {
    const filtered = movies?.value.filter(m => m.title.toLowerCase().includes(q.trim().toLowerCase()))

    displayMovies.value = filtered
  } else {
    displayMovies.value = movies?.value
  }
})

/**
 * Open add/edit movie dialog and set the form value if the method has argument of id on it.
 */
const showFormDialog = (id?: number | null) => {
  if (id) {
    const movie = movies?.value.filter(m => m.id === id)
    const selectedGenre: string[] = []

    movie![0].genres.forEach((data: string) => selectedGenre.push(data))

    formData.id = id
    formData.title = movie![0].title
    formData.director = movie![0].director
    formData.summary = movie![0].summary
    formData.genres = selectedGenre
  }

  formDialog.value = true
}

/**
 * Close the add/edit movie dialog and reset the formData to it's default value.
 */
const closeFormDialog = () => {
  formData.id = null
  formData.title = ''
  formData.director = ''
  formData.summary = ''
  formData.genres = []
  formMessage.genres = ''

  formDialog.value = false
}

/**
 * Method to mutate add or edit movie when the form is submitted and then close the dialog.
 */
const handleMutateMovie = () => {
  if (formData.id) {
    const index = movies?.value.findIndex(m => m.id === formData.id)
    const movie = movies?.value[index!]

    if (formMessage.genres) return false

    movie!.title = formData.title
    movie!.director = formData.director
    movie!.summary = formData.summary
    movie!.genres = formData.genres
  } else {
    const moviesID = movies?.value.map(m => m.id)
    const maxID = Math.max(...moviesID as [])

    if (formMessage.genres) return false

    movies?.value.push({
      id: maxID + 1,
      title: formData.title,
      director: formData.director,
      summary: formData.summary,
      genres: formData.genres
    })
  }

  closeFormDialog()
}

/**
 * Delete the movie and then close the dialog.
 */
const handleDeleteMovie = () => {
  const index = movies?.value.findIndex(m => m.id === formData.id)

  movies?.value.splice(index!, 1)

  closeFormDialog()
}

/**
 * Validate non quasar form, in this case the genre chips that acts like selections,
 * which can't be validated using quasar form validation (AFAIK). :v
 *
 * Therefore the genres are validated separately.
 */
const validateGenres = () => {
  if (!formData.genres.length) {
    formMessage.genres = 'Atleast one genres must be selected'
  } else {
    formMessage.genres = ''
  }
}

const toggleGenre = (genre: string) => {
  if (!formData.genres.includes(genre)) {
    formData.genres.push(genre)
  } else {
    const index = formData.genres.findIndex(g => g === genre)

    formData.genres.splice(index, 1)
  }

  validateGenres()
}
</script>

<template>
  <q-page padding>
    <q-input
      v-model="searchQuery"
      placeholder="Search movie title"
      debounce="250"
      outlined
    />

    <!-- List of Movies -->
    <div class="movies">
      <q-card
        :key="movie.title"
        v-for="movie in displayMovies"
        class="movies-card"
        bordered
        @click="showFormDialog(movie.id)"
      >
        <h2 class="movies-card__title text-h5">{{ movie.title }}</h2>
        <h3 class="movies-card__director text-subtitle2">{{ movie.director }}</h3>
        <div class="movies-card__genres">
          <q-chip
            :key="genre"
            v-for="genre in movie.genres"
            size="sm"
            color="primary"
            text-color="white"
            outline
          >
            {{ genre }}
          </q-chip>
        </div>
      </q-card>
    </div>

    <!-- FAB -->
    <q-page-sticky position="bottom-right" :offset="[18, 18]">
      <q-btn fab icon="add" color="primary" @click="() => showFormDialog()" />
    </q-page-sticky>

    <!-- Form Dialog -->
    <q-dialog
      v-model="formDialog"
      persistent
      maximized
      transition-show="slide-up"
      transition-hide="slide-down"
    >
      <q-card>
        <div class="bg-primary text-white">
          <q-toolbar>
            <q-toolbar-title>{{ formData.id ? 'Edit Movie' : 'Add Movie' }}</q-toolbar-title>
              <q-btn class="q-mr-md" flat type="submit" form="add-form" @click="validateGenres">Save</q-btn>
              <q-btn class="q-mr-md" flat v-if="formData.id" @click="handleDeleteMovie">Delete</q-btn>
              <q-btn flat icon="close" v-close-popup @click="closeFormDialog" />
          </q-toolbar>
        </div>
        <q-card-section>
          <q-form
            id="add-form"
            class="movies-form"
            @submit="handleMutateMovie"
            greedy
          >
            <q-input
              v-model="formData.title"
              placeholder="Title"
              outlined
              :rules="[(val: string) => val && val.length > 0 || 'Title must not be empty']"
            />
            <q-input
              v-model="formData.director"
              placeholder="Director"
              outlined
              :rules="[(val: string) => val && val.length > 0 || 'Director must not be empty']"
            />
            <q-input
              v-model="formData.summary"
              type="textarea"
              placeholder="Summary"
              outlined
              maxlength="100"
              hint="Maximum number of characters is 100"
              :rules="[(val: string) => val && val.length > 0 || 'Summary must not be empty']"
            />
            <div class="movies-form-field" :data-error="formMessage.genres ? true : undefined">
              <h5 class="movies-form-field__title text-subtitle1">Genres</h5>
              <div class="movies-form__genres">
                <q-chip
                  :key="genre"
                  v-for="genre in genres"
                  :color="formData.genres.includes(genre) ? 'primary' : undefined"
                  :text-color="formData.genres.includes(genre) ? 'white' : undefined"
                  clickable
                  @click="() => toggleGenre(genre)"
                >
                  {{ genre }}
                </q-chip>
              </div>
              <p class="movies-form-field__hint" v-if="formMessage.genres">{{ formMessage.genres }}</p>
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<style lang="scss">
.brx-header {
  width: 100%;
  min-height: 50px;
  padding: 0 12px;
}

.movies {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-top: 20px;

  &-card {
    cursor: pointer;
    padding: 14px 16px;
    box-shadow: none;
    transition: box-shadow 200ms cubic-bezier(0.63, 0.01, 0.29, 1);

    &:hover {
      box-shadow: 0px 2px 6px rgba(127, 90, 255, 0.1);
    }

    &__title {
      margin: 0 0 4px;
    }

    &__director {
      margin: 0;
    }

    &__genres {
      display: flex;
      gap: 8px;
      margin-top: 16px;

      .q-chip {
        margin: 0;
      }
    }
  }

  &-form {
    display: flex;
    flex-direction: column;
    gap: 16px;

    &-field {
      &__title {
        margin-top: 0;
        margin-bottom: 8px;
      }

      &__hint {
        color: rgba(0, 0, 0, 0.54);
        font-size: 12px;
        line-height: 1;
        padding: 8px 12px 0;
        margin-bottom: 0;
      }

      &[data-error] .movies-form-field__hint {
        color: var(--q-negative);
      }
    }

    &__genres {
      display: flex;
      gap: 8px;

      .q-chip {
        margin: 0;
      }
    }
  }
}
</style>
