<script setup lang="ts">
import { onMounted, ref } from 'vue';

const allCards = ['1', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A'];
const playerCards = ref<string[]>([])
const playerCardsValue = ref(0);

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
    A: [10,1]
})
function drawTwoFirstCards() {
    const firstCard = getRandomCard();
    const secondCard = getRandomCard();

    playerCards.value = [firstCard, secondCard];

    playerCardsValue.value = countCardValue(playerCards);
}

function getRandomCard() {
    return allCards[Math.floor(Math.random() * allCards.length)];
}

function countCardValue(playerCards: object) {
    let cardsValue : number = 0;

    playerCards.value.forEach((item : string) => {
        cardsValue += cardValues.value[item][0];
    })

    return cardsValue;
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
        </div>
    </div>
</template>

<style scoped></style>
