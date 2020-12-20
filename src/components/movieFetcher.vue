<template>
    <div>
        Enter a movie to add:
        <input type="text" v-model="movieName"/>
        <button @click="addMovie" type="submit">Submit</button>

    </div>
</template>

<script>
    export default {
        name: "MovieFetcher",
        data() {
            return {
                movieName: ''
            }
        },
        methods: {
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
            }
        },
    }
</script>

<style lang="scss" scoped>

</style>