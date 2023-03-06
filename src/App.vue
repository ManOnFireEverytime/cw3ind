<template>
  <div id="app">
    <nav class="navbar navbar-expand-lg navbar-light bg-light fixed-top">
      <div class="container px-4 px-lg-5">
        <div class="form-group">
          <input
            type="text"
            v-model="search"
            placeholder="Search"
            class="search form-control"
          />
        </div>

        <div class="form-group">
          <div id="sort-bar">
            <select
              class="sort btn btn-default"
              name="sortBy"
              id="select"
              v-model="sortBy"
            >
              <option value="subject">Subject</option>
              <option value="price">Price</option>
              <option value="location">Location</option>
              <option value="space">Spaces</option>
            </select>
            <button
              v-on:click="ascending = !ascending"
              class="sort btn btn-default"
            >
              <a class="sort btn btn-default" v-if="ascending">Ascending</a>
              <a class="sort btn btn-default" v-else>Descending</a>
            </button>
          </div>
        </div>
        <button
        v-if="this.cart.length > 0"
          class="btn btn-outline-dark"
          v-on:click="changePage"
        >
          Cart
          <span class="badge bg-dark text-white ms-1 rounded-pill">{{
            cartItemCount
          }}</span>
        </button>
        <button v-else class="btn btn-outline-dark disabled">Cart</button>
      </div>
    </nav>

    <main>
      <component
        :is="currentView"
        :filteredLessons="filteredLessons"
        :urls="urls"
        :cart="cart"
        @add-item-to-cart="addToCart"
        @remove-item-from-cart="removeFromCart"
      ></component>
    </main>
  </div>
</template>

<script>
import Lesson from "./components/Lesson.vue";
import Checkout from "./components/Checkout.vue";

export default {
  name: "app",
  data() {
    return {
      currentView: Lesson,
      lessons: [],
      cart: [],
      urls: "https://webstoreapp-env.eba-yi2fch33.eu-west-2.elasticbeanstalk.com/",
      ascending: true,
      sortBy: "subject",
      search: "",
    };
  },
  components: {
    Lesson,
    Checkout,
  },

  created: function () {
    if ("serviceWorker" in navigator) {
      navigator.serviceWorker.register("service-worker.js");
    }
    fetch(
      "https://webstoreapp-env.eba-yi2fch33.eu-west-2.elasticbeanstalk.com/collections/lessons"
    )
      .then((response) => response.json())
      .then((lessons) => {
        this.lessons = lessons;
      });
  },
  methods: {
    changePage() {
      if (this.currentView == Lesson) {
        this.currentView = Checkout;
      } else {
        this.currentView = Lesson;
      }
    },

    // adds a lesson to cart
    addToCart(lessonId) {
      // find selected lesson id
      var lesson = this.getLessonById(lessonId);
      if (lesson.space > 0) {
        // decrease lesson space
        --lesson.space;

        // get existing item from cart
        var itemInCart = this.getCartItemFromCartByLessonId(lessonId);
        if (itemInCart != null) {
          // update existing item in cart
          ++itemInCart.space;
        } else {
          // adding new item to cart
          itemInCart = {
            lessonId: lessonId,
            space: 1,
            lesson: lesson,
            // name: "Ayo",
            // phoneNumber: "099497329987",
          };

          // update lesson space with put
          this.cart.push(itemInCart);
        }
      }
    },
    // removes a lesson from cart
    removeFromCart(lessonId) {
      // find selected lesson in cart
      var itemInCart = this.getCartItemFromCartByLessonId(lessonId);

      if (itemInCart.space == 1) {
        // if just one item space is left, remove item completely from cart
        var index = this.cart.map((x) => x.lessonId).indexOf(lessonId);
        this.cart.splice(index, 1);

        // redirect user back to home if cart is empty
        if (this.cart.length <= 0) {
          this.changePage();
        }
      } else {
        // reduce number of spaces of item in cart
        --itemInCart.space;
      }

      // increase lesson space
      var lesson = this.getLessonById(lessonId);
      ++lesson.space;
    },
    // get lesson by id
    getLessonById(lessonId) {
      var lesson = this.lessons.find((u) => u.id == lessonId);
      return lesson;
    },
    // get item in cart by id
    getCartItemFromCartByLessonId(lessonId) {
      var item = this.cart.find((u) => u.lessonId == lessonId);
      return item;
    },
  },
  computed: {
    //Searching & Sorting
    filteredLessons() {
      let tempLessons = this.lessons;

      //Search Function
      if (this.search != "") {
        tempLessons = tempLessons.filter((item) => {
          return (
            item.topic.toLowerCase().match(this.search.toLowerCase()) ||
            item.location.toLowerCase().match(this.search.toLowerCase())
          );
        });
      }

      //Sorting Function
      tempLessons = tempLessons.sort((a, b) => {
        //Sorting by subject
        if (this.sortBy == "subject") {
          let fa = a.topic.toLowerCase(),
            fb = b.topic.toLowerCase();

          if (fa < fb) {
            return -1;
          }
          if (fa > fb) {
            return 1;
          }
          return 0;
        }
        //Sorting by price
        else if (this.sortBy == "price") {
          return a.price - b.price;
        }
        //Sorting by location
        else if (this.sortBy == "location") {
          let fa = a.location.toLowerCase(),
            fb = b.location.toLowerCase();

          if (fa < fb) {
            return -1;
          }
          if (fa > fb) {
            return 1;
          }
          return 0;
        }
        //Sorting by space
        else if (this.sortBy == "space") {
          let x = a.space;
          let y = b.space;
          if (x < y) {
            return 1;
          }
          if (x > y) {
            return -1;
          }
          return 0;
        }
      });
      //Descending order
      if (!this.ascending) {
        tempLessons.reverse();
      }

      return tempLessons;
    },
    // Number of items in Cart
    cartItemCount: function () {
      if (this.cart.length > 0)
        return this.cart.reduce((total, item) => total + item.space, 0);
      return 0;
    },
  },
};
</script>
