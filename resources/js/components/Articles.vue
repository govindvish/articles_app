<template>
    <div>
        <h2 class="my-5">Articles</h2>
        <div class="mb-3">
            <form @submit.prevent="addArticle">
                <div class="form-group">
                    <input type="text" class="form-control" placeholder="Title" v-model="article.title">
                </div>
                <div class="form-group">
                    <textarea class="form-control" placeholder="Body" v-model="article.body"></textarea>
                </div>
                <button type="submit" class="btn btn-light btn-block">Save</button>
            </form>
        </div>
        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li v-bind:class="[{disabled: !pagination.prev_page_url}]" class="page-item"><a class="page-link" href="#" @click="fetchArticles(pagination.prev_page_url)">Previous</a></li>
                <li class="page-item disabled"><a class="page-link text-dark" href="#">Page {{pagination.current_page}} of {{pagination.last_page}}</a></li>
                <li v-bind:class="[{disabled: !pagination.next_page_url}]" class="page-item"><a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">Next</a></li>
            </ul>
        </nav>
        <div class="card card-body mb-2" v-for="article in articles" v-bind:key="article.id">
            <h3>{{article.title}}</h3>
            <p>{{article.body}}</p>
            <hr>
            <button class="btn btn-warning mb-2" @click="editArticle(article)">Edit</button>
            <button class="btn btn-danger" @click="deleteArticle(article.id)">Delete</button>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            articles: [],
            article: {
                id: '',
                title: '',
                body: ''
            },
            article_id: '',
            pagination: {},
            edit: false
        }
    },

    created() {
        this.fetchArticles();
    },
     
    methods: {
        fetchArticles(page_url) {
            let vm = this;
            page_url = page_url || 'api/articles'
            axios.get(page_url)
                .then(res => {
                    this.articles = res.data.data;
                    vm.makePagination(res.data.meta, res.data.links);
                })
                .catch(err => console.log(err));
        },
        makePagination(meta, links) {
            let pagination = {
                current_page: meta.current_page,
                last_page: meta.last_page,
                next_page_url: links.next,
                prev_page_url: links.prev
            }

            this.pagination = pagination;
        },
        deleteArticle(id) {
            if(confirm('Are you sure?')) {
                axios.delete(`api/article/${id}`)
                    .then(res => {
                        alert('Article Removed');
                        this.fetchArticles();
                    })
                    .catch(err => console.log(err));
            }
        },
        addArticle() {
            if(this.edit === false) {
                // Add
                axios({
                    method: 'POST',
                    url: 'api/article',
                    data: JSON.stringify(this.article),
                    headers: {
                        'content-type': 'application/json'
                    }
                })
                    .then(res => {
                        this.article.title = '';
                        this.article.body = '';
                        alert('Article added');
                        this.fetchArticles();
                    })
                    .catch(err => console.log(err))
            } else {
                // Update
                axios({
                    method: 'PUT',
                    url: 'api/article',
                    data: JSON.stringify(this.article),
                    headers: {
                        'content-type': 'application/json'
                    }
                })
                    .then(res => {
                        this.article.title = '';
                        this.article.body = '';
                        alert('Article Updated');
                        this.fetchArticles();
                    })
                    .catch(err => console.log(err))
            }
        },
        editArticle(article) {
            this.edit = true;
            this.article.id = article.id;
            this.article.article_id = article.id;
            this.article.title = article.title;
            this.article.body = article.body;
        }
    }
}
</script>