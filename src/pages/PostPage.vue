<template>
    <div >
      <h1>Сторінка з постами</h1>
  
      <my-input v-focus v-model="searchQuery" placeholder="Пошук....." />
      <div class="app__btns">
        <my-button @click="showDialog">Створити пост</my-button>
        <my-select v-model="selectedSort" :options="sortOptions" />
      </div>
      <my-dialog v-model:show="dialogVisible">
        <post-form @create="createPost" />
      </my-dialog>
  
      <post-list
        :posts="sortedAndSearchedPosts"
        @remove="removePost"
        v-if="!isPostLoading"
      />
      <div v-else>Завантажується ...</div>
        <div 

        v-intersection="loadMorePosts"
        class="observer"
        >
  
        </div>
  
      <!-- <div class="page__wrapper">
        <div 
        v-for="pageNumber in totalPages" 
        :key="pageNumber"
        class="page"
        :class="{
          'current-page': page === pageNumber
        }"
        @click="changePage(pageNumber)"
        >{{ pageNumber }}</div>
  
      </div> -->
    </div>
  </template>
  
  <script>
  import PostForm from "@/components/PostForm.vue";
  import PostList from "@/components/PostList.vue";
  import MyButton from "@/components/UI/MyButton.vue";
  import MySelect from "@/components/UI/MySelect.vue";
  import axios from "axios";
  export default {
    components: { PostForm, PostList, MyButton, MySelect },
    data() {
      return {
        posts: [
          // { id: 1, title: "Vue", body: "About Vue" },
          // { id: 2, title: "React", body: "About React" },
          // { id: 3, title: "Angular", body: "About Angular" },
          // { id: 4, title: "Node.js", body: "About Node" },
        ],
        dialogVisible: false,
        isPostLoading: false,
        page: 1,
        limit: 10,
        totalPages: 0,
        searchQuery: "",
        selectedSort: "",
        sortOptions: [
          { value: "title", name: "По названию" },
          { value: "body", name: "По содержимому" },
        ],
      };
    },
    methods: {
      createPost(post) {
        this.posts.push(post);
        this.dialogVisible = false;
      },
      removePost(post) {
        this.posts = this.posts.filter((p) => p.id !== post.id);
      },
      showDialog() {
        this.dialogVisible = true;
      },
      // changePage(pageNumber){
      //       this.page=pageNumber
  
      // },
      async fetchPosts() {
        try {
          this.isPostLoading = true;
          const response = await axios.get(
            "https://jsonplaceholder.typicode.com/posts",
            {
              params: {
                _page: this.page,
                _limit: this.limit,
              },
            }
          );
          this.totalPages = Math.ceil(
            response.headers["x-total-count"] / this.limit
          );
          this.posts = response.data;
          // console.log(response);
        } catch (err) {
          alert("Помилка");
          console.error(err);
          this.error = "Не вдалося завантажити пости. Спробуйте ще раз пізніше.";
        } finally {
          this.isPostLoading = false;
        }
      },
      async loadMorePosts() {
        try {
          this.page+=1
         
          const response = await axios.get(
            "https://jsonplaceholder.typicode.com/posts",
            {
              params: {
                _page: this.page,
                _limit: this.limit,
              },
            }
          );
          this.totalPages = Math.ceil(
            response.headers["x-total-count"] / this.limit
          );
          this.posts = [...this.posts , ...response.data];
         
        } catch (err) {
          alert("Помилка");
          console.error(err);
          this.error = "Не вдалося завантажити пости. Спробуйте ще раз пізніше.";
        } 
      },
  
    },
    mounted() {
      this.fetchPosts();
    
  //     const options = {
   
  //     rootMargin: '0px',
  //     threshold: 1.0
  // }
  // const callback = (entries, observer)=> {
  //     if(entries[0].isIntersecting && this.page<this.totalPages){
  //       this.loadMorePosts()
  //     }
  // };
  // const observer = new IntersectionObserver(callback, options);
  // observer.observe(this.$refs.observer)
    },
  
    computed: {
      sortedPosts() {
        return [...this.posts].sort((post1, post2) =>
          post1[this.selectedSort]?.localeCompare(post2[this.selectedSort])
        );
      },
      sortedAndSearchedPosts() {
        return this.sortedPosts.filter((post) =>
          post.title.toLowerCase().includes(this.searchQuery.toLocaleLowerCase())
        );
      },
    },
  
    watch: {
      // selectedSort(newValue){
      //   this.posts.sort((post1 , post2)=>{
      //   return post1[newValue]?.localeCompare(post2[newValue])
      //   })
      // },
      // page(){
      //   this.fetchPosts()
      // }
    },
  };
  </script>
  
  <style>

  
  .app__btns {
    margin: 12px 0;
    display: flex;
    justify-content: space-between;
  }
  .page__wrapper {
    display: flex;
    margin-top: 15px;
  }
  .page {
    border: 1px solid black;
    padding: 10px;
  }
  .current-page {
    border: 2px solid teal;
  }
  .observer {
    height: 30px;
    
  }
  </style>
  