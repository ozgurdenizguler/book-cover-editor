<template>
  <div class="choose-book">
    <select class="select-book" @change="selectBook($event.target.value)" v-model="selectedBookTitle">
      <option disabled>Select a book</option>
      <option v-for="book in books" :key="book.title">{{ book.title }} - {{ book.author }}</option>
    </select>
  </div>
</template>

<script>
export default {
  name: 'ChooseBook',
  props: {
    books: Array
  },
  data() {
    return {
      selectedBookTitle: ""
    };
  },
  mounted() {
    const storedBookTitle = localStorage.getItem("selectedBookTitle");
    if (storedBookTitle) {
      this.selectedBookTitle = storedBookTitle;
    }
  },
  methods: {
    selectBook(selectedBookTitle) {
      const match = /(.*) - (.*)/.exec(selectedBookTitle);
      if (match) {
        const title = match[1]; 
        const author = match[2]; 
        const selectedBook = { title, author };
        localStorage.setItem("selectedBookTitle", selectedBookTitle);
        this.$emit('bookSelected', selectedBook);
      } else {
        console.error('Invalid book format');
      }
    }
  }
}
</script>

<style scoped>
.choose-book {
  margin-bottom: 20px;
}

.select-book {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
  outline: none;
  transition: border-color 0.3s ease;
}

.select-book:hover,
.select-book:focus {
  border-color: #007bff;
}

.select-book option {
  padding: 5px;
}

</style>
