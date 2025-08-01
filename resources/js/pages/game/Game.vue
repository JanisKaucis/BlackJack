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
const playerBalance = ref(100);
const playerBet = ref(100);
const minBet = 5;
const maxBet = ref(100);
const canPlay = ref(true);
const betStep = 5;

const cardValues = ref<Record<string, array>>({
    1: 1,
    2: 2,
    3: 3,
    4: 4,
    5: 5,
    6: 6,
    7: 7,
    8: 8,
    9: 9,
    10: 10,
    J: 10,
    Q: 10,
    K: 10,
    A: 11,
});

function drawTwoFirstCards() {
    const firstCard = getRandomCard();
    const secondCard = getRandomCard();

    playerCards.value = [firstCard, secondCard];

    playerCardsValue.value = countCardValue(playerCards);
}

async function playerDrawACard() {
    playerCards.value.push(getRandomCard());
    playerCardsValue.value = countCardValue(playerCards);

    playerLose.value = checkIfLose(playerCardsValue);

    if (playerLose.value) {
        playerBalance.value -= playerBet.value;
        checkIfBalanceEnoughToPlay();
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

    if (dealerLose.value) {
        playerBalance.value += playerBet.value;
    }

    if (dealerCardsValue.value <= 21 && dealerCardsValue.value > playerCardsValue.value) {
        playerLose.value = true;
        playerBalance.value -= playerBet.value;
    } else if (dealerCardsValue.value <= 21 && dealerCardsValue.value < playerCardsValue.value) {
        dealerLose.value = true;
        playerBalance.value += playerBet.value;
    } else if (dealerCardsValue.value === playerCardsValue.value) {
        itIsADraw.value = true;
    }
    checkIfBalanceEnoughToPlay();
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
    setMaxBetAvailable();
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
        cardsValue += cardValues.value[item];
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

function addBet() {
    if (playerBet.value >= maxBet.value) {
        return;
    }
    playerBet.value += betStep;
}

function reduceBet() {
    if (playerBet.value <= betStep) {
        return;
    }
    playerBet.value -= betStep;
}

function checkIfBalanceEnoughToPlay() {
    if (playerBalance.value < minBet) {
        canPlay.value = false;
        return;
    }

    canPlay.value = true;
}

function setMaxBetAvailable() {
    maxBet.value = playerBalance.value;

    if (playerBet.value > playerBalance.value) {
        playerBet.value = playerBalance.value;
    }
}

function sleep(ms: number) {
    return new Promise((resolve) => setTimeout(resolve, ms));
}

onMounted(() => {
    drawTwoFirstCards();
});
</script>

<template>
    <div class="flex h-screen flex-col">
        <div>
            <div>Balance: {{ playerBalance }}</div>
            <div>Bet: {{ playerBet }}</div>
            <div>
                <Button @click="addBet" class="m-1 p-2">+</Button>
                <Button @click="reduceBet" class="m-1 p-2">-</Button>
            </div>
        </div>
        <div class="flex h-1/2 items-center justify-center">
            <div class="flex">
                <div v-for="(card, index) in dealerCards" :key="index">
                    <div class="rounded-sm border bg-white p-4 text-black">{{ card }}</div>
                </div>
            </div>
        </div>
        <div v-if="playerLose" class="m-2 flex items-center justify-center">
            <div>
                <div>You lost!</div>
                <div v-if="!canPlay">No balance!</div>
            </div>
        </div>
        <div v-if="dealerLose" class="m-2 flex items-center justify-center">You win!</div>
        <div v-if="itIsADraw" class="m-2 flex items-center justify-center">It's a draw!</div>
        <div v-if="playAgain && canPlay" class="m-2 flex items-center justify-center">
            <Button @click="startNewGame">Play again</Button>
        </div>
        <div class="flex h-1/2 items-center justify-center">
            <div class="flex">
                <div v-for="(card, index) in playerCards" :key="index">
                    <div class="rounded-sm border bg-white p-4 text-black">{{ card }}</div>
                </div>
            </div>
            <div class="m-2 flex justify-between">
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
