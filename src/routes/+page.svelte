<script>
  import { onMount } from "svelte";
  import "../app.css";

  var learn = {};
  var quiz = {
    spanish: [0, 1, 2, 3],
  };
  var learnMode = true;
  var score = 0;
  var loading = true;

  onStart();

  async function onStart() {
    const res = await fetch(
      "https://server.nagendraallam.com/api/v1/get-random-word",
      {
        method: "GET",
        headers: {
          "Content-Type": "application/json",
        },
      }
    )
      .then((res) => {
        return res.json();
      })
      .then((data) => {
        learn = data;
      });

    // get randomnumber between 0 - 3000
    let random = Math.floor(Math.random() * 3000);

    const res2 = await fetch(
      "https://server.nagendraallam.com/api/v1/quiz?id=" + random,
      {
        method: "GET",
        headers: {
          "Content-Type": "application/json",
        },
      }
    )
      .then((res) => {
        return res.json();
      })
      .then((data) => {
        // shuffle data array
        data.spanish.sort(() => Math.random() - 0.5);
        quiz = data;
        loading = false;
      });
  }

  async function refresh() {
    await fetch("https://server.nagendraallam.com/api/v1/get-random-word", {
      method: "GET",
      headers: {
        "Content-Type": "application/json",
      },
    })
      .then((res) => {
        return res.json();
      })
      .then((data) => {
        learn = data;
      });
  }

  function changeMode() {
    learnMode = !learnMode;
  }

  function checkAnswer(e) {
    if (loading) return;
    // get value of button from e
    const value = e.target.outerText;
    if (e.target.outerText === quiz.correct) {
      score++;
      loading = true;
      document.getElementById("correct-ans").classList.remove("hidden");
      setTimeout(async () => {
        // get randomnumber between 0 - 3000
        let random = Math.floor(Math.random() * 3000);
        const res2 = await fetch(
          "https://server.nagendraallam.com/api/v1/quiz?id=" + random,
          {
            method: "GET",
            headers: {
              "Content-Type": "application/json",
            },
          }
        )
          .then((res) => {
            return res.json();
          })
          .then((data) => {
            // shuffle data array
            data.spanish.sort(() => Math.random() - 0.5);
            quiz = data;
            loading = false;
          });
        document.getElementById("correct-ans").classList.add("hidden");
      }, 2000);
    } else {
      score = 0;
      document.getElementById("wrong-ans").classList.remove("hidden");
      setTimeout(() => {
        document.getElementById("wrong-ans").classList.add("hidden");
      }, 2000);
    }
  }
</script>

<div
  class="h-screen w-screen bg-slate-700 text-white flex flex-col justify-center items-center"
>
  <div class="text-4xl font-bold underline mb-10 mt-10">
    {#if learnMode}
      <h1>Learn Spanish</h1>
    {:else}
      <h1>Take a quiz</h1>
    {/if}
  </div>

  <button
    class="fixed top-0 right-0 border-2 border-white pr-4 pl-4 pt-2 pb-2 mr-10 mt-4"
    on:click={changeMode}
  >
    {#if learnMode}
      Take Quiz
    {:else}
      Start Learning
    {/if}
  </button>
  {#if learnMode}
    <div class="flex flex-col text-center justify-center items-center mt-10">
      <div class="flex flex-row text-6xl w-[100vw] items-center justify-center">
        <p>{(learn && learn.english) || ""}</p>
        &nbsp; : &nbsp;
        <p>{(learn && learn.spanish) || ""}</p>
      </div>
      <button
        class="border-2 w-[10vw] text-2xl font-bold border-white pr-4 pl-4 pt-2 pb-2 mr-10 mt-16"
        on:click={refresh}>Next word</button
      >
    </div>
  {:else}
    <p class="text-2xl underline">Score: {score}</p>
    <p class="text-2xl font-bold">Select the correct spanish word for</p>
    <h1 class="text-4xl mb-10">&quot;{quiz.english || ""}&quot;</h1>
    <p id="wrong-ans" class="border-white border-2 p-4 mt-2 hidden bg-red-600">
      Wrong Answer! Try again
    </p>
    <p
      id="correct-ans"
      class="border-white border-2 p-4 mt-2 hidden bg-green-600"
    >
      Correct Answer!
    </p>
    <div class="grid grid-cols-2">
      {#each quiz.spanish as item}
        <button
          class="border-2 w-[20vw] text-2xl font-bold border-white pr-4 pl-4 pt-2 pb-2 m-2"
          on:click={checkAnswer}>{item}</button
        >
      {/each}
    </div>
  {/if}
</div>
