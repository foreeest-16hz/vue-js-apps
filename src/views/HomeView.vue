<script setup></script>
<template>
  <div class="container p-5">
    <div class="row align-items-center mb-5">
      <div class="col">
        <h3>View Picsum images</h3>
      </div>
      <div class="col-auto">
        <div class="btn-group">
          <button
            type="button"
            class="btn btn-primary dropdown-toggle"
            data-bs-toggle="dropdown"
            aria-expanded="false"
          >
            {{ filter.PageSize.Name }}
          </button>
          <ul class="dropdown-menu">
            <li
              class="dropdown-item"
              v-for="(size, index) in pageSizes"
              :key="size.Name"
              :class="{ 'dropdown-selected-item': filter.PageSize.Name === size.Name }"
              @click="getImages(size)"
            >
              {{ size.Name }}
            </li>
          </ul>
        </div>
      </div>
    </div>

    <div class="row">
      <div class="col-sm-3-custom mb-3" v-for="(img, index) in images" :key="img.id">
        <ImageSkeletonLoaderComponent skeleton-color="#dbdbdb" v-if="img.IsLoading" />
        <v-lazy-image :src="img.download_url" class="picsum-image" v-else />
      </div>
    </div>
    <div ref="imagesBottomRef" />

    <div class="row" v-if="imagesLoading">
      <div class="col-sm-3-custom mb-3" v-for="index in filter.PageSize.Value">
        <ImageSkeletonLoaderComponent skeleton-color="#dbdbdb" />
      </div>
    </div>
  </div>
</template>

<script>
import ImageSkeletonLoaderComponent from '@/components/loaders/ImageSkeletonLoaderComponent.vue'
import VLazyImage from 'v-lazy-image'

export default {
  components: {
    ImageSkeletonLoaderComponent,
    'v-lazy-image': VLazyImage,
  },
  data() {
    return {
      filter: {
        PageNumber: 1,
        PageSize: {
          Name: '5',
          Value: 6,
        },
      },
      pageSizes: [
        {
          Name: '5',
          Value: 6,
        },
        {
          Name: '10',
          Value: 10,
        },
        {
          Name: '25',
          Value: 25,
        },
        {
          Name: '50',
          Value: 50,
        },
        {
          Name: 'All',
          Value: 20,
        },
      ],

      images: [],
      imagesLoading: false,
    }
  },
  methods: {
    //#region helpers
    handleLoadImages(image) {
      image.IsLoading = true
      this.preloadImage(image.download_url)
        .then(() => {
          image.IsLoading = false
        })
        .catch((error) => {
          console.error('Image failed to preload', error)
          image.IsLoading = false
        })
    },
    preloadImage(url) {
      return new Promise((resolve, reject) => {
        const img = new Image()
        img.src = url
        img.onload = resolve
        img.onerror = reject
      })
    },
    //#endregion

    //#region API calls
    getImages(pageSize) {
      this.imagesLoading = true
      this.images = []
      if (pageSize) {
        this.filter.PageSize = pageSize
      }
      this.$axios
        .get(
          `https://picsum.photos/v2/list?page=${this.filter.PageNumber}&limit=${this.filter.PageSize.Value}`,
        )
        .then((response) => {
          this.images = response.data
          this.imagesLoading = false
          this.images.map((img) => this.handleLoadImages(img))
        })
        .catch((error) => {
          console.error(error)
          this.imagesLoading = false
        })
    },
    showMoreImages() {
      this.imagesLoading = true
      this.$axios
        .get(
          `https://picsum.photos/v2/list?page=${this.filter.PageNumber}&limit=${this.filter.PageSize.Value}`,
        )
        .then((response) => {
          this.images = [...this.images, ...response.data]
          this.imagesLoading = false
          this.images.map((img) => this.handleLoadImages(img))
        })
        .catch((error) => {
          console.error(error)
          this.imagesLoading = false
        })
    },
    //#endregion

    userScrolledToBottomOfList() {
      if (this.filter.PageSize.Name !== 'All') return

      const item = this.$refs.imagesBottomRef
      if (!item) return

      const options = {
        threshold: 0.5,
      }

      const observer = new IntersectionObserver((entries, observer) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            if (!this.imagesLoading) {
              this.filter.PageNumber += 1
              this.showMoreImages()
            }
          }
        })
      }, options)
      observer.observe(item)
    },
  },
  updated() {
    this.userScrolledToBottomOfList()
  },
  created() {
    this.getImages()
  },
}
</script>
<style scoped>
@media (min-width: 500px) {
  .col-sm-3-custom {
    flex: 0 0 auto;
    width: 33.33333333%;
  }
}

.picsum-image {
  min-height: 100px;
  max-height: 400px;
  height: 20vh;
  width: 100%;
  object-fit: cover;
  border-radius: 10px;
}

@media (min-width: 992px) {
  .picsum-image {
    height: 400px;
  }
}
</style>
