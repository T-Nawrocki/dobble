<template>
  <div id="game-board">
      <div id="ui-components">
        <timer />
        <div id="score-display">Score: <span class="score-span">{{score}}</span></div>
        <button id="btn-game-board" class="btn" v-on:click="mainMenu">Main menu</button>
      </div>
      <img class="logo" src="../assets/dobble.png" alt="dobble-logo"/>
      <div class="card-div">
        <card :card="dealtLeftCard" class="card"/>
        <card :card="dealtRightCard" class="card"/>
      </div>
  </div>
</template>

<script>
import CardService from '../services/CardService.js'
import Timer from './Timer.vue'
import Card from './Card.vue'
import { eventBus } from '@/main.js'
import { shuffle } from "../helpers/ShuffleArray";

export default {
    name: 'game-board',

    data() {
        return {
            cards: [],
            dealtLeftCard: null,
            dealtRightCard: null,
            selectedSymbols: [],
            score: 0
        }
    },

    mounted() {
        // retrieves cards from database, 
        // shuffles cards' symbols, 
        // then deals two cards
        CardService.getCards()
            .then(cards => this.cards = cards)
            .then(() => this.dealCards());

        // from Card: pushes symbol into selectedSymbols array, then checks if guess is correct
        // if guess is correct calls winRound, else if there are two selectedSymbols calls incorrectGuess 
        eventBus.$on('symbol-selected', (cardSymbol) => { 
            this.selectedSymbols.push(cardSymbol);
            if (this.checkWin()) {
                this.winRound();
            } else if (this.twoSymbols && !this.symbolsSame) {
                this.incorrectGuess();
            }
        });

        // from Card: changes selectedSymbols to an array containing the symbol that has been chosen
        // will only be called if there is exactly one symbol in the array (symbol-selected in all other cases)
        eventBus.$on("symbol-changed", cardSymbol => {
            this.selectedSymbols = [cardSymbol]
        });

        // from Timer: sends new game-over event bus (containing score) to App
        eventBus.$on("time-up", () => {
            eventBus.$emit("game-over", this.score)
        })

        // clean values when game ends
        eventBus.$on("game-over", () => {
            this.cards = [];
            this.dealtLeftCard = null;
            this.dealtRightCard = null;
            this.selectedSymbols = [];
        });

        eventBus.$on("main-menu", () => {
            this.cards = [];
            this.dealtLeftCard = null;
            this.dealtRightCard = null;
            this.selectedSymbols = [];
        });

    },

    beforeDestroy () {
        eventBus.$off('symbol-selected');
    },

    computed: {

        twoSymbols() {
            return this.selectedSymbols.length === 2;
        },

        symbolsSame() {
            return this.twoSymbols && this.selectedSymbols[0] === this.selectedSymbols[1];
        }

    },

    methods: {

        // TODO REEMOVE
        checkWin() {
            return this.twoSymbols && this.symbolsSame;
        },

        mainMenu () {
            eventBus.$emit("main-menu");  // to App
        },

        // selects a random card object from the array of cards, then sets that value to dealtLeftCard
        dealLeftCard() {
            const card = this.cards[Math.floor(Math.random() * this.cards.length)];
            this.dealtLeftCard = card;
        },

        // selcts a random card object from the array of cards
        // if the content of that object is the same as the content of dealtLeftCard, repeats the process
        // else, sets dealtRightCard to the selected card
        dealRightCard() {
            const card = this.cards[Math.floor(Math.random() * this.cards.length)];
            if (JSON.stringify(card) === JSON.stringify(this.dealtLeftCard) ) {
                this.dealRightCard();
            } else {
                this.dealtRightCard = card;
            }
        },

        // empties selectedSymbols array then sends eventBus to Card (card then deselects symbol)
        incorrectGuess() {
            this.selectedSymbols = [];
            eventBus.$emit('guess-over');
        },

        // increments score
        // then empties selectedSymbols array
        // then deals two new cards
        // finally sends eventBus to Card (card then deselects symbol)
        winRound() {
            eventBus.$emit('guess-over');
            this.score += 1;
            this.selectedSymbols = [];
            this.dealCards();
        },

        // deals cards and then shuffles symbols
        dealCards() {
            this.dealLeftCard();
            this.dealRightCard();
            shuffle(this.dealtLeftCard.symbols);
            shuffle(this.dealtRightCard.symbols);
        }

    },
    
    components: {
        "timer": Timer,
        "card": Card
    }
    
}    
</script>

<style>
    #game-board {
        width: 90vw;
        height: 85vh;
        background: url('../assets/wood-pattern.png') #996100;
        background-position: center;
        background-size: auto;
        border-radius: 2rem;
    }
    .logo {
        position: absolute;
        top: 22%;
        left: 50%;
        transform: translate(-50%, -22%);
        width: 80px;
        height: 80px;
    }
    .card-div {
        display: flex;
        justify-content: space-evenly;
        align-items: center;
        transform: translateY(20vh);
    }
    .card {
        display: inline-block;
        width: 150px;
        height: 150px;
        background-color: #f4f4f4;
        border-radius: 50%;
        border: 1px solid rgba(124, 1, 124, 0.6);
        box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.75);
    }
    .card:hover {
        box-shadow: 0px 0px 25px 5px rgba(0,0,0,0.75);
        background-color: white;
        transition: 0.3s;
    }
    #ui-components {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        justify-items: center;
        align-items: center;
        margin: 0 auto;
    }
    #btn-game-board {
        width: 12.5vw;
        max-width: 420px;

    }
    
    @media screen and (min-width: 1000px) {
        .card {
            width: 250px;
            height: 250px;
        }
        #score-display, .score-span {
            font-size: 30px;
        }
        #timer, .time-limit-span {
            font-size: 30px;
        }
    }
</style>
