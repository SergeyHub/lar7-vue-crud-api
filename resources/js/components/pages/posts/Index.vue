<template>
    <div>
        <h1>All Posts</h1>

        <div class="row">
            <div class="col-sm-6">

                <validation-errors :errors="validationErrors"
                           v-if="validationErrors"></validation-errors>

                <form @submit.prevent="addPost" class="mb-4">
                    <div class="form-group">
                        <label for="title">Title</label>
                        <input v-model="post.title" type="text" class="form-control" id="title">
                    </div>

                    <div class="form-group">
                        <label for="description">Post</label>
                        <textarea v-model="post.description" class="form-control" id="description" rows="3"></textarea>
                    </div>
                    
                    <button type="submit" class="btn btn-primary">Save</button>
                    <button v-if="edit" @click="edit = false"
                            type="button" class="btn btn-light">Cancel</button>
                </form>
            </div>
        </div>

        <div v-if="errored" class="alert alert-danger" role="alert">
            The entries did not load, please try again later :
        </div>

        <table v-else class="table table-striped">
            <div v-if="loading">Loading...</div>
            <thead>
                <tr>
                    <th scope="col">ID</th>
                    <th scope="col">Title</th>
                    <th scope="col">Post</th>
                    <th scope="col"></th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="post in posts" :key="post.id">
                    <th>{{ post.id }}</th>
                    <td>{{ post.title }}</td>
                    <td>{{ post.description }}</td>
                    <td>
                        <button @click="editPost(post)" class="btn btn-success">
                            <i class="fa fa-pencil"></i><br>
                        </button><br>
                        <button @click="deletePost(post.id)" class="btn btn-danger">
                            <i class="fa fa-trash"></i>
                        </button>
                    </td>
                </tr>
            </tbody>
        </table>

        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li :class="{ disabled: !pagination.prev_page_url }"
                    @click.prevent="getPosts(pagination.prev_page_url)" 
                    class="page-item">
                    <a class="page-link" href="#">Previous</a>
                </li>
                <li class="page-item disabled">
                    <a class="page-link" href="#">
                        Page {{ pagination.current_page }} of {{ pagination.last_page }}
                    </a>
                </li>
                <li :class="{ disabled: !pagination.next_page_url }"
                    @click.prevent="getPosts(pagination.next_page_url)"
                    class="page-item">
                    <a class="page-link" href="#">Next</a>
                </li>
            </ul>
        </nav>

    </div>
</template>

<script>
export default {
    data() {
        return {
            posts: [],
            post: {
                id: '',
                title: '',
                description: ''
            },
            pagination: {},
            edit: false,
            loading: true,
            errored: false,
            validationErrors: ''
        }
    },
    mounted() {
        this.getPosts()
    },
    methods: {
        getPosts(page_url) {
            page_url = page_url || '/api/posts'
            
            axios
                .get(page_url)
                .then(response => {
                    this.posts = response.data.data
                    this.makePagination(response.data)
                })
                .catch(error => {
                    console.log(error)
                    this.errored = true;
                })
                .finally(() => this.loading = false)
        },
        makePagination(response) {
            let pagination = {
                current_page: response.current_page,
                last_page: response.last_page,
                prev_page_url: response.prev_page_url,
                next_page_url: response.next_page_url
            }

            this.pagination = pagination
        },
        deletePost(id) {
            axios
               .delete(`/api/posts/${id}`)
               .then(response => {
                   this.getPosts()
                   console.log(response)
                })
               .catch(error => console.log(error))
        },
        addPost() {
            if (this.edit === false) {
                //  add post
                axios
                  .post('/api/posts', {
                      title: this.post.title,
                      description: this.post.description
                  })
                  .then(response => {
                      this.post.title = ''
                      this.post.description = ''
                      this.getPosts()

                      console.log(response)
                  })
                  .catch(error => {
                      if (error.response.status === 422) {
                         this.validationErrors = error.response.data.errors;
                      }
                      
                      console.log(error)
                  })
            } else {
                //  edit post
                axios
                  .put(`/api/posts/${this.post.id}`, {
                      title: this.post.title,
                      description: this.post.description
                  })
                  .then(response => {
                      this.post.title = ''
                      this.post.description = ''
                      this.edit = false
                      this.getPosts()

                      console.log(response)
                  })
                  .catch(error => {
                      if (error.response.status === 422) {
                         this.validationErrors = error.response.data.errors;
                      }

                      console.log(error)
                  })
            }
        },
        editPost(post) {
            this.edit = true
            this.post.id = post.id
            this.post.title = post.title
            this.post.description = post.description
        }
    }
}
</script>