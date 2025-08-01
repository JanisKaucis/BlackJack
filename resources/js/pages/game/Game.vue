<script setup lang="ts">
import { Button } from '@/components/ui/button';
import { onMounted, ref } from 'vue';

const allCards = ['1', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A'];
const playerCards = ref<string[]>([]);
const playerCardsValue = ref(0);
let lose = false;

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

function drawACard() {
    playerCards.value.push(getRandomCard());
    playerCardsValue.value = countCardValue(playerCards);
    console.log(playerCardsValue.value);

    lose = checkIfLose(playerCardsValue)
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

function checkIfLose(playerCardsValue :object) {
    return playerCardsValue.value > 21;
}

onMounted(() => {
    drawTwoFirstCards();
});
</script>

<template>
    <div class="flex h-screen flex-col">
        <div class="flex h-1/2 items-center justify-center">dealer cards</div>
        <div class="flex h-1/2 items-center justify-center">
            <div class="flex">
                <div v-for="(card, index) in playerCards" :key="index">
                    <div class="rounded-sm border bg-white p-4 text-black">{{ card }}</div>
                </div>
            </div>
            <div class="m-2" v-if="!lose">
                <Button v-on:click="drawACard">Draw a Card</Button>
            </div>
            <div v-if="lose">
                <p>You lost!</p>
            </div>
        </div>
    </div>
</template>

<style scoped></style>
