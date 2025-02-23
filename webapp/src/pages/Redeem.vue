<script setup>

import { ref, onMounted, watch } from 'vue';
import GLOBALS from '../globals.js'
import Footer from '../components/Footer.vue'

const props = defineProps(['closeDetailScreen', 'detailNFT', 'setScratched', 'toggleModal'])

let count = ref(8);
let imageLoaded = ref(false)
let nftAddress = GLOBALS.NFT_ADDRESS

watch(count, async (newValue)=> {
    if (newValue == 0) {
        const duration = 3 * 1000,
        animationEnd = Date.now() + duration,
        defaults = { startVelocity: 30, spread: 360, ticks: 60, zIndex: 0 };

        function randomInRange(min, max) {
        return Math.random() * (max - min) + min;
        }

        const interval = setInterval(function() {
        const timeLeft = animationEnd - Date.now();

        if (timeLeft <= 0) {
            return clearInterval(interval);
        }

        const particleCount = 50 * (timeLeft / duration);

        confetti(
                Object.assign({}, defaults, {
                particleCount,
                origin: { x: randomInRange(0.1, 0.3), y: Math.random() - 0.2 },
                })
            );
            confetti(
                Object.assign({}, defaults, {
                particleCount,
                origin: { x: randomInRange(0.7, 0.9), y: Math.random() - 0.2 },
                })
            );
            
        }, 200)
    }

})

onMounted(() => {


    const img = new Image();
    img.src = `https://scratchverse.s3.us-west-1.amazonaws.com/${props.detailNFT.id}/${nftAddress}.jpg`
    img.onload = () => {
        setupScratch()
        imageLoaded.value = true;       
    };

    function setupScratch() {
        let scratched = [false, false, false, false, false, false, false, false];
        var options1 = { id: 'scratchcanvas1', brushSize: 10, lineJoin: 'round', percentRequired: 10, fillColor: '#bdbdbd' };
        var options2 = { id: 'scratchcanvas2', brushSize: 10, lineJoin: 'round', percentRequired: 10, fillColor: '#bdbdbd' };
        var options3 = { id: 'scratchcanvas3', brushSize: 10, lineJoin: 'round', percentRequired: 10, fillColor: '#bdbdbd' };
        var options4 = { id: 'scratchcanvas4', brushSize: 10, lineJoin: 'round', percentRequired: 10, fillColor: '#bdbdbd' };
        var options5 = { id: 'scratchcanvas5', brushSize: 10, lineJoin: 'round', percentRequired: 10, fillColor: '#bdbdbd' };
        var options6 = { id: 'scratchcanvas6', brushSize: 10, lineJoin: 'round', percentRequired: 10, fillColor: '#bdbdbd' };
        var options7 = { id: 'scratchcanvas7', brushSize: 10, lineJoin: 'round', percentRequired: 10, fillColor: '#bdbdbd' };
        var options8 = { id: 'scratchcanvas8', brushSize: 10, lineJoin: 'round', percentRequired: 10, fillColor: '#bdbdbd' };
        let one = new window.ScratchCard(options1);
        let two = new window.ScratchCard(options2);
        let three = new window.ScratchCard(options3);
        let four = new window.ScratchCard(options4);
        let five = new window.ScratchCard(options5);
        let six = new window.ScratchCard(options6);
        let seven = new window.ScratchCard(options7);
        let eight = new window.ScratchCard(options8);

        let arr = [one, two, three, four, five, six, seven, eight]
        arr.forEach((item, idx) => {
            item.addEventListener('success', function (e) {
                if(scratched[idx] == false) {
                    scratched[idx] = true
                    count.value--;
                }
                if(count.value == 0) {
                    props.setScratched(props.detailNFT.id);
                }
            }, false);
        })
    }

})
</script>

<template>
    <div class="page">
        <div class="left">
            <div class="btn-holder">
                <a style="cursor: pointer" @click="closeDetailScreen()"><h5 class="breadcrumb"><i class="fa-solid fa-arrow-left"></i> Return to Ticket Overview</h5></a>
                <h2><span style="color: #fac43b">Scratch 3 matching numbers to win</span></h2>
                <h3>Fields left to scratch: {{ count }}</h3>
                <p style="color: white; font-weight: 500;">Scratch the tickets by dragging your mouse over the scratch fields. You can redeem your winnings after the fields have been scratched. 
                </p>
            </div>
        </div>
        <div class="cont" id="conthandler">
            <div v-if="!imageLoaded" style="padding: 100px;">
                <div style="text-align: center;">
                    <div class="lds-ring"><div></div><div></div><div></div><div></div></div>
                </div>
            </div>
            <div class="ticketholder animate__animated animate__backInDown " v-show="imageLoaded" :style="{'background-image': `url(https://scratchverse.s3.us-west-1.amazonaws.com/${detailNFT.id}/${nftAddress}.jpg)` } ">
                <canvas id="scratchcanvas1" width="69" height="69"></canvas>
                <canvas id="scratchcanvas2" width="69" height="69"></canvas>
                <canvas id="scratchcanvas3" width="69" height="69"></canvas>
                <canvas id="scratchcanvas4" width="69" height="69"></canvas>
                <canvas id="scratchcanvas5" width="69" height="69"></canvas>
                <canvas id="scratchcanvas6" width="69" height="69"></canvas>
                <canvas id="scratchcanvas7" width="69" height="69"></canvas>
                <canvas id="scratchcanvas8" width="69" height="69"></canvas>
            </div>
        </div>

        <div class="right" v-if="count == 0">
            <h2 class="win">You won a prize!</h2>
            <h3 class="win"> {{ detailNFT.prize }} Verse</h3>
            <p class="claim">Congratulations! You can claim your prize instantly</p>
            <button class="btn btn-redeem" style="cursor: pointer" v-if="detailNFT.claimed == false" @click="toggleModal(detailNFT.id)" >Claim Now</button>
            <button href="/tickets" class="btn btn-redeem" style="cursor: pointer" v-if="detailNFT.claimed == true" @click="closeDetailScreen()">Back to overview</button>
        </div>
        <Footer />
    </div>
</template>


<style lang="scss">
.right {
    @media(max-width: 880px) {
        position: fixed;
        background-color: #1a1833;
        padding: 20px;
        bottom: 0;
        left: 0;
        z-index: 3;
        border-top: 1px solid white;
    }
    padding-top: 130px;
    .claim {
        color: white;
        margin-top: 0;
        margin-bottom: 0;
    }
    padding-left: 50px;
    h2.win {
        color: white!important;
    }
    h3.win {
        color: #fac43b!important;
        margin-bottom: 0;
    }
    float: left;
}
.breadcrumb {
    color: white;
}
h2.win {
    margin-bottom: 0;
    font-size: 17px;
   
}
h3.win {
    margin-top: 3px;
    color: #ffc700!important;
    font-size: 40px;
}
.left {
    @media(max-width: 880px) {
       padding: 20px;
       width: calc(100% - 40px);
       margin-top: 0;
       padding-top: 0;
    }
    margin-top: 50px;
    width: 32.7%;
    float: left;
    padding: 40px;
}
.btn-holder {
    h2 {
        color: white;
        text-align: left;
    }
    h3 {
        color: white;
    }
    p {
        color: white;
    }
}

.btn-check {
    background-color: #68686861;
    color: white;
    border: none;
    height: 40px;
    margin-right: 20px;
    padding: 10px 20px;
    border-radius: 5px;
    margin-top: 10px;
}

.btn-redeem {
    background: radial-gradient(circle farthest-corner at 10% 20%, rgb(249, 232, 51) 0%, rgb(250, 196, 59) 100.2%);
    color: #333;
    font-size: 15px;
    font-weight: 600;
    border: none;
    height: 40px;
    padding: 10px 20px;
    border-radius: 5px;
    margin-top: 10px;
}

.cont {
    float: left;
    @media(max-width: 880px) {
        padding-top: 0;
        padding-left: 0;
        padding-bottom: 200px;
        width: 100%;
        overflow: auto;
    }
    padding-top: 25px;
    h2 {
        color: white;
        text-align: center;
    }
}
.ticketholder {
    position: relative;
    margin: 0 auto;
    background-size: contain;
    background-repeat: no-repeat;
    width: 356px;
    height: 720px;

}
#scratchcanvas1 {
    border-radius: 50%;
    position: absolute;
    bottom: 286px;
    left: 22px;
}

#scratchcanvas2 {
    border-radius: 50%;
    position: absolute;
    bottom: 286px;
    left: 103px;
}

#scratchcanvas3 {
    border-radius: 50%;
    position: absolute;
    bottom: 286px;
    left: 184px;
}

#scratchcanvas4 {
    border-radius: 50%;
    position: absolute;
    bottom: 286px;
    left: 264px;
}

#scratchcanvas5 {
    border-radius: 50%;
    position: absolute;
    bottom: 207px;
    left: 22px;
}

#scratchcanvas6 {
    border-radius: 50%;
    position: absolute;
    bottom: 207px;
    left: 103px;
}

#scratchcanvas7 {
    border-radius: 50%;
    position: absolute;
    bottom: 207px;
    left: 184px;
}

#scratchcanvas8 {
    border-radius: 50%;
    position: absolute;
    bottom: 207px;
    left: 264px;
}



</style>