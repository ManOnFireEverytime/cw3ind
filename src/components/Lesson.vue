<template>
  <section class="py-5">
          <div class="container px-4 px-lg-5 mt-5">
            <div
              class="row gx-4 gx-lg-5 row-cols-2 row-cols-md-3 row-cols-xl-4 justify-content-center"
            >
              <div
                v-for="item in filteredLessons"
                :key="item.id"
                class="col mb-5"
              >
                <div class="card h-100">
                  <img
                    v-bind:src="urls + item.image"
                    class="card-img-top"
                  />
                  <div class="card-body p-4">
                    <div class="text-center">
                      <h5 class="fw-bolder">Subject: {{ item.topic }}</h5>
                      <p><font-awesome-icon v-bind:icon="item.icon" /></p>
                      <p>Location: {{ item.location }}</p>
                      <p>Spaces: {{ item.space }}</p>
                      Price: £{{ item.price }}
                    </div>
                  </div>
                  <div class="card-footer p-4 pt-0 border-top-0 bg-transparent">
                    <div class="text-center" v-if="canAddToCart(item)">
                      <a
                        class="btn btn-outline-dark mt-auto"
                        v-on:click="addToCart(item.id)"
                        >Add to Cart</a
                      >
                    </div>

                    <div class="text-center" v-else>
                      <a class="btn btn-outline-dark mt-auto disabled"
                        >Out of stock</a
                      >
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </section>
</template>

<script>
export default {
  name: 'app',
  props: ["filteredLessons", "urls", "cart"],
  methods: {

    canAddToCart(item) {
      return item.space > 0;
    },
    // Item Cart count
    cartCount(item) {
      let count = 0;
      for (var i = 0; i < this.cart.length; i++) {
        if (this.cart[i] === item) {
          count++;
        }
      }
      return count;
    },
    // adds a lesson to cart
    addToCart: function(lessonId) {
      this.$emit("add-item-to-cart", lessonId)
    },
  },
}

</script>