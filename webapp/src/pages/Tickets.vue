<script>
import { getAccount, waitForTransaction, readContract, disconnect, writeContract, watchAccount, watchNetwork } from '@wagmi/core'
import { ref } from 'vue';
import ERC721ABI from '../abi/ERC721.json'
import Redeem from '../pages/Redeem.vue'
import { useWeb3Modal } from '@web3modal/wagmi/vue'
import ContractABI from '../abi/contract.json'
import ERC721 from '../abi/ERC721.json'
import { useRoute } from 'vue-router'
import GLOBALS from '../globals.js'
import Footer from '../components/Footer.vue'

export default {
    components: {
        Redeem,
        Footer
    },  
    setup() {
        const route = useRoute()
        const contractAddress = GLOBALS.CONTRACT_ADDRESS
        const nftContract = GLOBALS.NFT_ADDRESS

        let list = []
        let account = getAccount()
        let accountActive = ref(false)
        let loading = ref(false)
        let modal = useWeb3Modal()

        let giftModal = ref(false)
        let giftAccount = ref("")
        let claimNFT = ref(0)
        let step = ref(0);
        let nfts = ref([])
        let claimActive = ref(false)
        let modalLoading = ref(false)

        let openDetail = ref(false);
        let detailNFT = ref({});


        if(route.query.gift && route.query.address && route.query.gift.length > 0 && route.query.address.length > 0) {
            disconnect()
            giftModal.value = true
            giftAccount.value = route.query.address
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
        }, 250);
        }

        watchAccount(async () => {
            if(getAccount().address &&  getAccount().address.length != undefined) {

                accountActive.value = true;
                getTicketIds()

            } else {
                accountActive.value = false
            }
        })

        async function redeem(nftId) {
            modalLoading.value = true
            const obj = nfts.value.find(obj => obj.id == nftId);
            console.log(obj)

            try {
            const { hash } = await writeContract({
            address: contractAddress,
            abi: ContractABI,
            functionName: 'claimPrize',
            chainId: 137,
            args: [obj.id]
            })
            await waitForTransaction({ hash })
            modalLoading.value = false
            const objToUpdate = nfts.value.find(obj => obj.id == nftId);
            objToUpdate.claimed = true
            step.value = 1;

            } catch (e) {
                modalLoading.value = false;
                console.log(e)
            }

        }

        function setScratched(id) {
            localStorage.setItem(id.toString() + '/' + nftContract.toString(), true)
            const objToUpdate = nfts.value.find(obj => obj.id === id);

            if (objToUpdate) {
                objToUpdate.scratched = true;
            }

        }

        function openDetailScreen(id) {
            detailNFT.value = nfts.value.find(obj => obj.id === id);
            openDetail.value = true;
        }

        function closeGiftModal(connect) {
            if(connect) {
                modal.open()
            }
            giftModal.value = false
        }


        async function getClaimed(id) {
            try {
                const data = await readContract({
                address: GLOBALS.NFT_ADDRESS,
                abi: ERC721,
                functionName: 'claimed',
                args: [id]
                })
                
                if(data) {
                    const objToUpdate = nfts.value.find(obj => obj.id == id);
                    if (objToUpdate) {
                        objToUpdate.claimed = data;
                        
                    }
                    return data 
                }
            } catch (e) {
                console.log(e)
            }
               
        }


        async function getEdition(id) {
            try {
                const data = await readContract({
                address: GLOBALS.NFT_ADDRESS,
                abi: ERC721,
                functionName: 'editions',
                args: [id]
                })
                if(data) {
                    const objToUpdate = nfts.value.find(obj => obj.id == id);
                    if (objToUpdate) {
                        objToUpdate.edition = parseInt(data);
                    }
                    return data 
                }
            } catch (e) {
                console.log(e)
            }
               
        }

        async function getPrizeAmount(id) {
            try {
                const data = await readContract({
                address: GLOBALS.NFT_ADDRESS,
                abi: ERC721,
                functionName: 'prizes',
                args: [id]
                })
                if(data) {
                    const objToUpdate = nfts.value.find(obj => obj.id == id);
                    if (objToUpdate) {
                        objToUpdate.prize = parseInt(data);
                    }
                    return data 
                }
            } catch (e) {
                console.log(e)
            }
               
        }


        function toggleModal(id) {
            step.value = 0;
            if(id) claimNFT.value = nfts.value.find(obj => obj.id === id);
            claimActive.value = !claimActive.value
        }

        async function getRedemptionStatus(id) {
            try {
                const data = await readContract({
                address: nftContract,
                abi: ERC721ABI,
                functionName: 'tokenURI',
                args: [id]
                })
                if(data) {
                    const objToUpdate = nfts.value.find(obj => obj.id == id);
                    // let edition = parseInt(data.split("&edition=")[1])
                    // objToUpdate.edition = edition
                    if(data.includes("/true")) {
                        if (objToUpdate) {
                            objToUpdate.claimed = true;
                        } else {
                            objToUpdate.claimed = false
                        }
                    }
                    return data
                }
            } catch (e) {
                console.log(e)
            }
        }

        function ticketList() {
            return nfts.value.toReversed()
        }


        function closeDetailScreen() {
            openDetail.value = false;
        }
        async function getTicketIds() {
            try {
            // step 1, check balance
            loading.value = true;
            const data = await readContract({
            address: nftContract,
            abi: ERC721ABI,
            functionName: 'ownedByAddress',
            args: [getAccount().address]
            })


            /// step 2, check allowance 
      
            if(data) {
                let promiseArray = []
                 let arr = []
                 data.forEach(dat => {
                    let scratched = false
                    if(localStorage.getItem(dat.toString() + '/' + nftContract.toString()) == 'true') {
                        scratched = true
                    }
                    arr.push({id: parseInt(dat.toString()), scratched, claimed: false })
                    promiseArray.push(getRedemptionStatus(dat.toString()))
                })
                nfts.value = arr

                nfts.value.forEach(nft => {
                    promiseArray.push(promiseArray.push(getClaimed(nft.id)))
                    promiseArray.push(promiseArray.push(getEdition(nft.id)))
                    promiseArray.push(promiseArray.push(getPrizeAmount(nft.id)))
                })
                await Promise.all(promiseArray)
                 loading.value = false;
            }
            } catch (e) {
                console.log(e)
                loading.value = false;
            }
        }   

        return {
            list, nfts, account, nftContract, closeGiftModal, step, loading, giftModal, giftAccount, claimNFT, claimActive, modalLoading, toggleModal, accountActive, getTicketIds, ticketList, openDetail, openDetailScreen, closeDetailScreen, detailNFT, setScratched, redeem, getRedemptionStatus
        }   
    }
}
</script>

<template>
    <div class="backdrop" v-if="claimActive || giftModal">
        <!-- gift-->
        <div class="modal" v-if="giftModal">
            <div class="modal-head">
                <p class="iholder"><i @click="closeGiftModal(false)" class="close-btn" ></i></p>
            </div>
            <div class="modal-body short">
                <div class="img-gift"></div>
                <h3 class="title">Gift Ticket Received!</h3>
                <p class="subtext">Somebody has sent a scratch ticket to you. Your ticket has a chance to win <span>100.000 Verse!</span>
                <br><br>No transaction needed to scratch. Connect your account (<span> {{ giftAccount.slice(0, 7) }}..</span>) to redeem the ticket.
                </p>
                
                <a @click="closeGiftModal(true)" v-if="accountActive == false"><button class="btn verse-wide fixBottomMobile">Connect and Redeem</button></a>
                <a @click="closeGiftModal(false)" v-if="accountActive == true"><button class="btn verse-wide fixBottomMobile">Redeem</button></a>
                <img url="/gift.png">
            </div>
        </div>
        
        <!-- claim-->
        <div class="modal" v-if="claimActive">
            <p class="iholder"><i @click="toggleModal()" class="fa fa-times"></i></p>
            <div v-if="modalLoading">
                <p style="text-align: center">claiming prize..</p>
                <div style="text-align: center;">
                    <div class="lds-ring"><div></div><div></div><div></div><div></div></div>
                </div>
            </div>
            <div v-if="!modalLoading && step == 0">
                <h3>Congrats on your win!</h3>
                <p>Funds will immediately be in your wallet after the transaction has been completed.</p>
                <a @click="redeem(claimNFT.id)"><button class="btn btn-modal verse">Claim {{claimNFT.prize }} Verse</button></a>
            </div>
            <div v-if="!modalLoading && step == 1">
                <h3>Successfully Claimed Win</h3>
                <p>Thanks for playing!</p>
                <a href="/"><button class="btn btn-modal verse">Buy new Ticket</button></a>
                <a href="/"><button class="btn btn-modal x">Gift a Ticket</button></a>
            </div>
        </div>
    </div>

<Redeem v-if="openDetail" :toggleModal="toggleModal" :closeDetailScreen="closeDetailScreen" :detailNFT="detailNFT" :setScratched="setScratched"/>
<div class="page" v-if="!openDetail">
    <div class="head">
        <h2 class="tickhead">My Tickets</h2>
        <div class="tickconnect" v-if="!accountActive">Connect your wallet to view your tickets. </div>
    </div>

    <div class="tickets" v-if="accountActive && loading">
        <div class="spin" >
            <div class="lds-ring"><div></div><div></div><div></div><div></div></div>
         </div>
    </div>
    
    <div class="tickets" v-if="accountActive && !loading">
        <div v-if="nfts.length == 0">
            <h3>Couldn't find any tickets in your connected wallet. Click <a href="/" style="text-decoration: none; font-weight: 600; color: rgb(250, 196, 59);">here</a> to buy tickets </h3>
        </div>
        <div class="ticket" v-for="item, index in ticketList()">
            <h3 class="title">Ticket {{item.id}} </h3>

            <p class="status" v-if="item.claimed == false">
                <i v-if="item.scratched" class="fa-solid fa-crown" style="color: gold;"></i> Claimable Ticket
            </p>
            <p class="status" v-if="item.claimed == true">
                <i class="fa-solid fa-crown" style="color: gold"></i> Claimed Ticket
            </p>

            <div v-if="item.claimed == false">
                <img style="height: 490px" class="mobreset" v-if="item.scratched == false" :src="'/prescratch/' + item.edition + '.png'">
                <img style="height: 490px" class="mobreset" v-if="item.scratched == true" :src="`https://scratchverse.s3.us-west-1.amazonaws.com/${item.id}/${nftContract}.jpg`">
            </div>

            
            <div v-if="item.claimed == true">
                <img :src="`https://scratchverse.s3.us-west-1.amazonaws.com/${item.id}/${nftContract}.jpg`">
            </div>

            <button v-if="item.scratched == false && item.claimed == false" class="btn-action main" @click="openDetailScreen(item.id)">Scratch Ticket</button>
            <button v-if="item.scratched == true && item.claimed == false" @click="toggleModal(item.id)" class="btn-action main" >Claim {{item.prize}} Verse</button>
            <button v-if="item.claimed == true" class="btn-action main dis" >{{item.prize}} Verse Claimed</button>
        </div>
    </div>
    <Footer v-if="!loading" />
</div>
</template>



<style lang="scss">

.mobreset {
    @media(max-width: 880px) {
        height: unset!important;
    }
}
.spin {
    width: 50px;
    padding-left: calc(50% - 50px);
    @media(max-width: 880px) {
        text-align: center!important;
    }
}
.title {
    margin-bottom: 0;
}
.status {
    font-size: 13px;
    margin-top: 0;
}
.btn-action {
    cursor: pointer;
    margin-top: 10px;
    border-radius: 5px;
    background-color: white;
    color: black;
    border: none;
    margin-right: 5px;
    font-weight: 500;
    font-size: 15px;
    padding: 13px 25px;
    font-weight: 600;

    &.main {
        width: 100%;
        color: #333;
        font-weight: 600;
        background-image: radial-gradient(circle farthest-corner at 10% 20%, rgb(51 249 238) 0%, rgb(19 255 179) 100.2%);
        background: radial-gradient(circle farthest-corner at 10% 20%, rgb(249, 232, 51) 0%, rgb(250, 196, 59) 100.2%);
    }

    &.dis {
        background-color: #353535;
        background-image: none;
        color: white;
    }
}

.tickconnect {
    margin-top: 50px;
    text-align: center;
}
.tickhead {
    @media(max-width: 880px) {
        display: none;
    }
}

.btn-modal {
    cursor: pointer;
    margin-top: 10px;
    border-radius: 5px;
    background-color: white;
    color: black;
    border: none;
    margin-right: 5px;
    font-weight: 500;
    font-size: 15px;
    padding: 13px 25px;
    font-weight: 600;

    &.x {
        background-color: #222;
        color: white;
        height: 47px;
        margin-left: 5px;
    }

    &.verse {
        color: #333;
        font-weight: 600;
        background-image: radial-gradient(circle farthest-corner at 10% 20%, rgb(51 249 238) 0%, rgb(19 255 179) 100.2%);
        background: radial-gradient(circle farthest-corner at 10% 20%, rgb(249, 232, 51) 0%, rgb(250, 196, 59) 100.2%);
    }
    
    &.uniswap {
        background-color: white!important;
        color: #1c1b22;
    }
}


div.tickets {
    width: calc(100% - 100px);
    display: inline-block;
    margin-bottom: 100px;
    padding-left: 100px;
    @media(max-width: 880px) {
        width: calc(100% - 10px);
        display: inline-block;
        padding-left: 10px;
        padding-top: 20px;
        margin-bottom: 200px!important;
    }
    h3 {
        font-weight: 400;
        color: white;
    }
    .ticket {
        @media(max-width: 880px) {
            width: 90%;
            margin-left: 3%;
        }
        position: static;
        color: white;
        display: inherit;
        width: 260px;

        margin-right: 10px;
        img {
            width: 100%;
        }
    }
}
.head {
    @media(max-width: 880px) {
       text-align : center;
       padding-left: 0;
    }
    padding-left: 100px;
    color: white;
}
.page {
    position: relative;
    width: 100%;
    height: calc(100vh - 70px);
    padding-left: 0;
    overflow: scroll;
}

</style>