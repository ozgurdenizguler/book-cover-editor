<template>
  <div id="app" class="cover">
    <h1 v-if="step === steps.CHOOSE_BOOK">Book Cover Editor</h1>
    <h1 v-else>Edit Cover</h1>
    <div v-if="step === steps.CHOOSE_BOOK">
      <h2>Choose Book</h2>
      <v-container>
        <ChooseBook :books="books" @bookSelected="bookSelectedHandler" />
      </v-container>
      <div class="button-container">
        <v-btn color="primary" @click="nextStep" :disabled="!selectedBook"
          >Next</v-btn
        >
      </div>
    </div>
    <div v-else-if="step === steps.COVER_UPLOAD">
      <v-container>
        <CoverUpload :selectedBook="selectedBook" @backClicked="prevStep" />
      </v-container>
    </div>
  </div>
</template>

<script>
import CoverUpload from "./components/CoverUpload.vue";
import axios from "axios";
import ChooseBook from "./components/ChooseBook.vue";

const STEPS = {
  CHOOSE_BOOK: "chooseBook",
  COVER_UPLOAD: "coverUpload",
};

export default {
  name: "App",
  components: {
    CoverUpload,
    ChooseBook,
  },
  data() {
    return {
      books: [],
      step: STEPS.CHOOSE_BOOK,
      selectedBook: null,
      steps: STEPS,
    };
  },
  mounted() {
    this.fetchBooks();
    const storedBookTitle = localStorage.getItem("selectedBookTitle");
    if (storedBookTitle) {
      const match = /(.*) - (.*)/.exec(storedBookTitle);
      if (match) {
        const title = match[1];
        const author = match[2];
        this.selectedBook = { title, author };
      }
    }
  },
  methods: {
    async fetchBooks() {
      try {
        const response = await axios.get(
          "https://api.nytimes.com/svc/books/v3/lists/current/hardcover-fiction.json?api-key=A6ohmOk376s3rKcRh7GLHvPhRfxZMcdx"
        );
        this.books = response.data.results.books.map((book) => ({
          title: book.title,
          author: book.author,
        }));
      } catch (error) {
        console.error("Error fetching books:", error);
      }
    },
    bookSelectedHandler(selectedBook) {
      this.selectedBook = selectedBook;
    },
    nextStep() {
      this.step = STEPS.COVER_UPLOAD;
    },
    prevStep() {
      this.step = STEPS.CHOOSE_BOOK;
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;
}

.cover {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.button-container {
  margin-top: 20px;
}

h1 {
  font-size: 28px;
  margin-bottom: 5px;
  color: #1023ee;
}

h2 {
  font-size: 24px;
  margin-top: 20px;
  margin-bottom: 10px;
  color: #ea0e50;
}

.v-container {
  margin-top: 5px;
}
</style>
