<script setup lang="ts">
import { Button } from '@/components/ui/button';
import { onMounted, ref } from 'vue';

const allCards = ['1', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A'];
const playerCards = ref<string[]>([]);
const dealerCards = ref<string[]>([]);
const playerCardsValue = ref(0);
const dealerCardsValue = ref(0);
const playerLose = ref(false);
const dealerLose = ref(false);
const itIsADraw = ref(false);
const playAgain = ref(false);
const stay = ref(false);

const cardValues = ref<Record<string, array>>({
    1: [1],
    2: [2],
    3: [3],
    4: [4],
    5: [5],
    6: [6],
    7: [7],
    8: [8],
    9: [9],
    10: [10],
    J: [10],
    Q: [10],
    K: [10],
    A: [11, 1],
});

function drawTwoFirstCards() {
    const firstCard = getRandomCard();
    const secondCard = getRandomCard();

    playerCards.value = [firstCard, secondCard];

    playerCardsValue.value = countCardValue(playerCards);
    console.log(playerCardsValue.value);
}

async function playerDrawACard() {
    playerCards.value.push(getRandomCard());
    playerCardsValue.value = countCardValue(playerCards);

    playerLose.value = checkIfLose(playerCardsValue);

    if (playerLose.value) {
        playAgain.value = true;
    }
}

async function drawDealerCards() {
    const firstCard = getRandomCard();
    const secondCard = getRandomCard();

    dealerCards.value = [firstCard, secondCard];

    dealerCardsValue.value = countCardValue(dealerCards);

    while (dealerCardsValue.value < playerCardsValue.value) {
        await sleep(1000);
        await dealerDrawACard();
    }

    if (dealerCardsValue.value <= 21 && dealerCardsValue.value > playerCardsValue.value) {
        playerLose.value = true;
    }else if (dealerCardsValue.value <= 21 && dealerCardsValue.value < playerCardsValue.value) {
        dealerLose.value = true;
    }else if (dealerCardsValue.value === playerCardsValue.value) {
        itIsADraw.value = true;
    }

    playAgain.value = true;
}

function startNewGame() {
    drawTwoFirstCards();
    dealerCards.value = [];
    dealerCardsValue.value = 0;
    playerLose.value = false;
    dealerLose.value = false;
    itIsADraw.value = false;
    playAgain.value = false;
    stay.value = false;
}
async function dealerDrawACard() {
    dealerCards.value.push(getRandomCard());
    dealerCardsValue.value = countCardValue(dealerCards);

    dealerLose.value = checkIfLose(dealerCardsValue);
}

function stayWithHand() {
    stay.value = true;

    drawDealerCards();
}

function getRandomCard() {
    return allCards[Math.floor(Math.random() * allCards.length)];
}

function countCardValue(playerCards: object) {
    let cardsValue: number = 0;
    let acesCount = 0;
    playerCards.value.forEach((item: string) => {
        cardsValue += cardValues.value[item][0];
        if (item === 'A') {
            acesCount++;
        }
    });

    for (let i = 0; i < acesCount; i++) {
        if (cardsValue > 21) {
            cardsValue -= 10;
        }
    }

    return cardsValue;
}

function checkIfLose(cardsValue: object) {
    return cardsValue.value > 21;
}

function sleep(ms :number) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

onMounted(() => {
    drawTwoFirstCards();
});
</script>

<template>
    <div class="flex h-screen flex-col">
        <div class="flex h-1/2 items-center justify-center">
            <div class="flex">
                <div v-for="(card, index) in dealerCards" :key="index">
                    <div class="rounded-sm border bg-white p-4 text-black">{{ card }}</div>
                </div>
            </div>
        </div>
        <div v-if="playerLose" class="flex justify-center items-center m-2">
            You lost!
        </div>
        <div v-if="dealerLose" class="flex justify-center items-center m-2">
            You win!
        </div>
        <div v-if="itIsADraw" class="flex justify-center items-center m-2">
            It's a draw!
        </div>
        <div v-if="playAgain" class="flex justify-center items-center m-2">
            <Button @click="startNewGame">Play again</Button>
        </div>
        <div class="flex h-1/2 items-center justify-center">
            <div class="flex">
                <div v-for="(card, index) in playerCards" :key="index">
                    <div class="rounded-sm border bg-white p-4 text-black">{{ card }}</div>
                </div>
            </div>
            <div class="flex justify-between m-2">
                <div class="m-2">
                    <Button :disabled="stay || playerLose" @click="playerDrawACard">Draw a card</Button>
                </div>
                <div class="m-2">
                    <Button :disabled="stay || playerLose" @click="stayWithHand">Stay</Button>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped></style>
