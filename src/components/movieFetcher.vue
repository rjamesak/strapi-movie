<template>
    <div>
        <form class="post-form">
            <div class="inputDiv">
                Enter a movie to add:
                <input type="text" v-model="postFormData.movieName"/>
            </div>

            <div class="inputDiv">
                <label for="poster-file">Upload a movie poster</label>
                <input type="file" name="poster-file" id="poster-file" @change="posterFileChosen">
            </div>

            <div class="inputDiv">
                <!-- <button @click.prevent="addMovie" type="submit">Submit</button> -->
                <button @click.prevent="addMovieWithImage" type="submit">Submit</button>
            </div>
        </form>

        <form class="get-form">Enter a movie to see an image:
            <input type="text" v-model="getMovieName">
            <button type="submit" @click.prevent="getMovie">Get Movie</button>
        </form>
        <img class="movie-poster"  v-if="this.posterUrl !== ''" :src="this.posterUrl"/>
    </div>
</template>

<script>
    export default {
        name: "MovieFetcher",
        data() {
            return {
                postFormData: {
                    movieName: '',
                    posterFile: {},
                    posterName: '',
                },
                getMovieName: '',
                headers: {'content-type': 'application/json'},
                posterUrl: '',
                categories: [],
                movies: []
            }
        },
        methods: {
            posterFileChosen(e) {
                console.log('changed: ', e)
                this.postFormData.posterFile = e.target.files[0]
                this.postFormData.posterName = e.target.files[0].name
            },
            checkStatus(resp) {
                if (resp.status >= 200 && resp.status < 300) {
                    console.log('good response')
                    return this.parseJSON(resp)
                }
                console.log('bad resp in checkStatus: ', resp)
            },
            parseJSON(resp) {
                return (resp.json ? resp.json() : resp)
            },
            addMovie() {
                console.log('title: ', this.movieName)
                fetch('http://localhost:1337/movies', {
                    method: 'POST',
                    headers: {
                        'content-type': 'application/json'
                    },
                    body: JSON.stringify({title: this.movieName})
                })
                .then(response => response.json())
                .then(data => console.log(data))
            },
            addMovieWithImage() {
                
                const formData = new FormData()
                const data = {'title' : this.postFormData.movieName}
                formData.append('data', JSON.stringify(data))
                formData.append('files.poster', this.postFormData.posterFile)
                const files = {'name' : this.postFormData.posterName}
                formData.append('files', JSON.stringify(files))
                for(let val of formData.entries()) {
                    console.log(val[0], ', ', val[1])
                }

                fetch('http://localhost:1337/movies', {
                    method: 'POST',
                    body: formData
                })
                .then(response => response.json())
                .then(data => console.log(data))
            },
            getMovie() {
                console.log('in getMovie')
                fetch(process.env.VUE_APP_STRAPI_URL + '/movies?title=' + this.getMovieName, {
                    method: 'GET', 
                    headers: this.headers
                })
                .then(response => this.checkStatus(response))
                .then(data => {
                    console.log('data: ', data)
                    // optional chaining to check if poster exists in data
                    // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining
                    if(data[0]?.poster[0]?.url) {
                        this.posterUrl = process.env.VUE_APP_STRAPI_URL + data[0].poster[0].url
                        console.log(data[0].poster[0].url)
                    }
                    else {
                        this.posterUrl = ''
                        console.log('no poster')}
                }
                )
            }
        },
        beforeMount() {
            //fetch categories
            fetch(process.env.VUE_APP_STRAPI_URL + '/categories', {
                method: 'GET',
                headers: this.headers
            })
            .then(response => {
                this.checkStatus(response)
                console.log('categories response: ', response)
            })
            .then(data => {
                console.log('categories data', data)
                this.categories = data
            })

            //fetch movies
            fetch(process.env.VUE_APP_STRAPI_URL + '/movies', {
                method: 'GET',
                headers: this.headers
            })
            .then(response => {
                console.log('movie response: ', response)
                this.checkStatus(response)
            })
            .then(data => {
                console.log('movie data: ', data)
                this.movies = data
            })
        }
    }
</script>

<style lang="css" scoped>
.post-form {
    display: flex;
    flex-direction: column;
    width: 50%;
    box-shadow: 5px 5px 5px rgba(0, 0, 0, 0.2);
    border-style: solid;
    border-width: 1px;
    padding: 10px;
    margin: auto;
}
.get-form {
    width: 50%;
    box-shadow: 5px 5px 5px rgba(0, 0, 0, 0.2);
    border-style: solid;
    border-width: 1px;
    padding: 10px;
    margin: auto;
    margin-top: 20px;
}
.movie-poster {
    margin: auto;
    margin-top: 20px;
    box-shadow: 5px 5px 5px rgba(0,0,0,0.2);
}
form div {
    margin: 20px;
}
button {
    padding:0.35em 1.2em;
    border:0.1em solid #ccc;
    margin:0.5em 0.3em 0.3em 0;
    border-radius:0.12em;
    box-sizing: border-box;
    font-family: 'Roboto';
    text-decoration:none;
    /* color:#FFFFFF; */
    text-align:center;
    transition: all 0.2s;
}

</style>