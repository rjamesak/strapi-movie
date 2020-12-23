<template>
    <div>
        <form class="post-form">
            <div>
                Enter a movie to add:
                <input type="text" v-model="postFormData.movieName"/>
            </div>

            <div>
                <label for="poster-file">choose a movie poster</label>
                <input type="file" name="poster-file" id="poster-file" @change="posterFileChosen">
            </div>

            <div>
                <!-- <button @click.prevent="addMovie" type="submit">Submit</button> -->
                <button @click.prevent="addMovieWithImage" type="submit">Submit with Image</button>
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
                apiAddress: 'http://localhost:1337',
                headers: {'content-type': 'application/json'},
                posterUrl: ''
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
                console.log(resp)
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
                // const formData = {
                //     "data" : {"title" : this.postFormData.movieName},
                //     "files" : {"name" : this.postFormData.posterName},
                //     "files.poster" : this.posterFile
                // }
                // const request = new XMLHttpRequest()
                const formData = new FormData()
                const data = {'title' : this.postFormData.movieName}
                formData.append('data', JSON.stringify(data))
                formData.append('files.poster', this.postFormData.posterFile)
                const files = {'name' : this.postFormData.posterName}
                formData.append('files', JSON.stringify(files))
                for(let val of formData.entries()) {
                    console.log(val[0], ', ', val[1])
                }

                // request.open('POST', 'http://localhost:1337/movies')
                // request.send(formData)

                fetch('http://localhost:1337/movies', {
                    method: 'POST',
                    // headers: {
                    //     'content-type': 'multipart/form-data'
                    // },
                    body: formData
                })
                .then(response => response.json())
                .then(data => console.log(data))
            },
            getMovie() {
                console.log('in getMovie')
                console.log(this.apiAddress)
                fetch(this.apiAddress + '/movies?title=' + this.getMovieName, {
                    method: 'GET', 
                    headers: this.headers
                })
                .then(response => this.checkStatus(response))
                .then(data => {
                    console.log('data: ', data)
                    // optional chaining to check if poster exists in data
                    // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining
                    if(data[0]?.poster[0]?.url) {
                        this.posterUrl = this.apiAddress + data[0].poster[0].url
                        console.log(data[0].poster[0].url)
                    }
                    else {
                        this.posterUrl = ''
                        console.log('no poster')}
                }
                )
            }
        },
    }
</script>

<style lang="css" scoped>
.post-form {
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

</style>