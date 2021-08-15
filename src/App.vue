<template>
    <div class="flex flex-col justify-center items-center">
        <div
            class="flex justify-center items-center  m-12 mb-80 w-1/2 space-x-1"
        >
            <p class="min-w-max">
                <a
                    href="#"
                    class="text-white font-bold bg-purpletheme rounded-8xl border-black border-4 p-1 px-2"
                    @click.prevent="randomQuote"
                    >get random</a
                >
            </p>
            <input
                type="text"
                name="search"
                @keyup="changed"
                v-model="authorSearchField"
                placeholder="Search for author..."
                class="placeholder-purpletheme placeholder-opacity-60 text-purpletheme w-full rounded-8xl bg-backgroundcolor border-black border-4 p-0.5 pl-2"
                ref="searchField"
            />
        </div>
        <div class="m-12">
            <ul
                v-if="authorSearchField.length > 3"
                class="flex justify-between align-middle text-xl text-white font-bold"
            >
                <li
                    v-for="(author, index) in searchedAuthors"
                    :key="index"
                    class="m-8 w-44 h-10 align-middle"
                >
                    <a href="#" @click.prevent="search(author)">{{ author }}</a>
                </li>
            </ul>
        </div>
        <div
            class="flex flex-row justify-between items-center w-2/3"
            v-if="quotes.length > 0"
        >
            <a
                href="#"
                class="h-20 w-20 bg-black m-8 rounded-xl rounded-tl-8xl rounded-bl-8xl"
                v-if="pageIndex > 0"
                @click.prevent="pageIndex--"
            ></a>
            <div
                class="bg-backgroundcolor text-white p-6 rounded-xl text-xl w-quotes border-black border-16"
            >
                <p class="mb-6">"{{ quotes[pageIndex].content }}"</p>
                <p class="text-purpletheme">~ {{ quotes[pageIndex].author }}</p>
            </div>
            <a
                href="#"
                class="h-20 w-20 bg-black m-8 rounded-xl rounded-tr-8xl rounded-br-8xl"
                v-if="pageIndex < quotes.length - 1"
                @click.prevent="pageIndex++"
            ></a>
        </div>
    </div>
</template>

<script>
import { onMounted, ref } from "vue";

export default {
    setup() {
        const authorSearchField = ref("");
        const searchedAuthors = ref([]);
        const quotes = ref([]);
        const pageIndex = ref(0);
        const searchField = ref(null);

        const fetchQuotesByName = async (authorName) => {
            const response = await fetch(
                `http://api.quotable.io/quotes?author=${authorName}`
            );

            const data = await response.json();

            return data;
        };

        const randomQuote = async () => {
            quotes.value = []
            pageIndex.value = 0

            const response = await fetch("http://api.quotable.io/random");
            const quote = await response.json();

            quotes.value.push(quote);
            
        }

        const search = async (author) => {
            const allQuotes = await fetchQuotesByName(author);

            quotes.value = allQuotes.results;
            pageIndex.value = 0;

            authorSearchField.value = "";
        };

        const changed = async (e) => {
            let authors = JSON.parse(localStorage.getItem("authors"));

            searchedAuthors.value = [];

            for (let i = 0; i < authors.length; i++) {
                if (
                    authors[i]
                        .toLowerCase()
                        .replace(/\s/g, "")
                        .includes(
                            e.target.value.toLowerCase().replace(/\s.*/g, "")
                        )
                ) {
                    searchedAuthors.value.push(authors[i]);
                }
            }
        };

        const fetchAuthors = async (authors, page = 1) => {
            const response = await fetch(
                `http://api.quotable.io/authors?limit=150&page=${page}`
            );

            const data = await response.json();

            for (let i = 0; i < data.results.length; i++) {
                authors.push(data.results[i].name);
            }

            if (data.page <= data.totalPages) {
                authors = await fetchAuthors(authors, data.page + 1);
            }

            return authors;
        };

        onMounted(async () => {
            searchField.value.focus();

            const response = await fetch("http://api.quotable.io/random");

            const startQuote = await response.json();

            quotes.value.push(startQuote);

            let authors = JSON.parse(localStorage.getItem("authors"));
            if (authors === null) {
                authors = [];
            }

            if (authors.length < 1) {
                authors = await fetchAuthors(authors);
                localStorage.setItem("authors", JSON.stringify(authors));
            }

        });

        return {
            changed,
            authorSearchField,
            searchedAuthors,
            search,
            quotes,
            pageIndex,
            searchField,
            randomQuote,
        };
    },
};
</script>

<style></style>
