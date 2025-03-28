<template>
    <div>
        <div class="w3-sidebar w3-bar-block w3-black w3-card" style="width:130px">
            <h5 class="w3-bar-item">VoteChain</h5>
            <button id="btnHome" class="w3-bar-item w3-button tablink" @click="openLink('btnHome', 'home')">Home</button>
            <button id="btnVote" class="w3-bar-item w3-button tablink" @click="openLink('btnVote', 'vote')">Vote</button>
            <button id="btnResults" class="w3-bar-item w3-button tablink" @click="openLink('btnResults', 'results')">Results</button>
            <button id="btnAccount" class="w3-bar-item w3-button tablink" @click="openLink('btnAccount', 'account')">Account</button>
        </div>
        <div id="contents" class="w3-container">

            <header class="w3-container w3-padding" id="myHeader">
                <div class="w3-center">
                    <h1 class="w3-xxxlarge w3-animate-left">Presidential Election</h1>
                </div>
            </header>
            <div class="w3-panel w3-pale-red w3-center w3-card-4 w3-padding" v-if="errorMessage.length > 0"> {{errorMessage}} </div>
            <div id="tabs">
            
                <div id="home" class="w3-container tab w3-animate-left w3-center" style="display:block">
                    <h3>Welcome <code>{{accounts[0]}}</code></h3>

                    <div v-if="accounts.length > 0" class="w3-margin-top">
                        <div class="w3-center">                            
                            <div v-if="hasVoted" class="w3-panel w3-pale-green w3-padding-large">Congratulations, your vote has been casted and counted.</div>
                            <div v-else>
                                <div v-if="votingEnabled" class="w3-center w3-panel w3-padding-large w3-pale-blue">
                                    <h4>Voting has been opened and you are eligible to vote.</h4>
                                    <div class="w3-padding">
                                        <button @click="openLink('btnVote', 'vote')" class="w3-button w3-blue w3-round">Click here to Vote</button>
                                    </div>
                                </div>
                                <div v-else class="w3-panel w3-padding w3-pale-yellow">
                                    <h4>You are eligible to vote but voting is NOT yet open.</h4>
                                    <div v-if="isChairperson" class="w3-panel">
                                        <button @click="openVoting()" class="w3-button w3-green w3-round" >Open Voting</button>
                                    </div>                                    
                                    <div v-else class="w3-padding">Please contact the chairperson and request to open the voting.</div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div v-else class="w3-center w3-panel w3-padding-large w3-pale-blue">                        
                        <div class="w3-padding">
                            In order to vote and/or view the results, you need to connect your metamask amount.
                            Please don't forget to disconnect after you have voted.
                        </div>
                        <div class="w3-padding">
                            <button  @click="connectToMetamask()" class="w3-button w3-green w3-round">Connect your Metamask Account</button>
                        </div>
                    </div>
                </div>

                <div id="vote" class="w3-container tab w3-animate-left w3-center" style="display:none">
                    <h2 v-if="accounts.length > 0" class="w3-panel w3-center">Candidates</h2>
                    <div v-else class="w3-padding"><button  @click="connectToMetamask()" class="w3-button w3-green w3-round">Connect your Metamask Account</button></div>
                    <div class="w3-row-padding w3-center w3-margin-top">
                        <div v-for="(candidate, index) in candidates" :key="index" class="w3-quarter">
                            <div class="w3-card w3-container" style="min-height:260px">
                                <h3>{{ candidate.name }}</h3>
                                <img :src="candidate.imageUrl" height="120"/>
                                <div class="w3-panel">
                                    <button v-if="!hasVoted && votingEnabled" @click="confirmVote(index)" class="w3-button w3-block w3-green">Vote</button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <div id="results" class="w3-container tab w3-animate-left w3-center" style="display:none">
                    <h2 v-if="accounts.length > 0" class="w3-panel w3-center">Results</h2>
                    <div v-else class="w3-padding"><button  @click="connectToMetamask()" class="w3-button w3-green w3-round">Connect your Metamask Account</button></div>
                    <div class="w3-row-padding w3-center w3-margin-top">
                        <div v-for="(candidate, index) in candidates" :key="index" class="w3-quarter">
                            <div class="w3-card w3-container" style="min-height:260px">
                                <h3>{{ candidate.name }}<span v-if="candidate.isWinner" class='w3-xlarge'><sup>&#128081;</sup></span></h3>
                                <img :src="candidate.imageUrl" height="120"/>
                                <div class="w3-panel">
                                    Votes: <span class="w3-badge w3-green w3-large">{{ candidate.voteCount }}</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                
                <div id="account" class="w3-container tab w3-animate-left w3-center" style="display:none">
                    <div class="w3-panel w3-padding w3-margin-top">
                        <div v-if="isChairperson">
                            <div class="w3-card-4 w3-padding-large w3-margin">Leading Candidate: {{ winner }}</div>
                            <div class="w3-card-4 w3-padding-large w3-margin">Number of Candidates: {{ candidatesLength }}</div>
                            <div class="w3-card-4 w3-padding-large w3-margin">Voting Opened: {{ votingEnabled }}</div>
                            <div class="w3-card-4 w3-padding-large w3-margin">Chairperson Address: {{ chairpersonAddress }}</div>
                            <div v-if="votingEnabled" class="w3-panel">
                                <button @click="closeVoting()" class="w3-button w3-black w3-round w3-block" >Close Voting</button>
                            </div>
                            <div v-else class="w3-panel">
                                <button @click="openVoting()" class="w3-button w3-blue w3-round w3-block" >Open Voting</button>
                            </div>
                        </div>
                        <div class="w3-panel">
                            <button v-if="accounts.length > 0"  @click="disconnect()" class="w3-button w3-red w3-round w3-block" >Disconnect Metamask Account</button>
                            <button v-else  @click="connectToMetamask()" class="w3-button w3-green w3-round w3-block">Connect your Metamask Account</button>
                        </div>
                    </div>
                </div>
            </div>
            <footer class="w3-container w3-padding-16 w3-center">
            <p>Powered by <a href="#">VoteChain</a> - a decentralized voting application built on blockchain.</p>
            </footer>
        </div>
    </div> 
</template>

<script>
import { ethers, utils } from "ethers";
import ballotAbi from "../contract-abis/votechain.json"

export default {
    data() {
        return {
            provider: null,
            contract: null,
            candidatesLength: 0,
            candidates: [],
            errorMessage: "",
            accounts: [],
            balance: 0,
            message: "",
            votingEnabled: false,
            hasVoted: false,
            chairperson: null,
            isChairperson: null,
            senderAddress: null,
            chairpersonAddress: null,
            winner: null
}
    },
    methods: {
        createContractInstance() {
            this.contract = new ethers.Contract('0x4094FBfc9003550F6DE7178F24cC100cf284A583', ballotAbi)
            this.contract = this.contract.connect(this.provider)
            this.getVotingStatus()
            this.getVoterHasVoted()
            this.getChairperson()
            this.getWinner()
            this.getCandidates()
            //this.checkIfChairperson()
        },
        async connectToMetamask() {
            this.provider = new ethers.providers.Web3Provider(window.ethereum);
            this.accounts = await this.provider.send('eth_requestAccounts', []);
            this.getBalance();
            this.createContractInstance();
        },
        async getCandidates() {
            this.candidatesLength = await this.contract.getCandidatesLength();
            this.candidates = [];
            for(var i = 0; i < this.candidatesLength; i++) {
                var candidate = await this.contract.candidates(i)
                var _winner = false;
                if(this.winner == candidate.name) {
                    //_badge = "<span class='w3-badge w3-red w3-large'><i class='fas fa-crown' style='font-size:48px;color:red'></i></span>"
                    _winner = true
                } else {
                    _winner = false
                }
                var _candidate = {
                    name: candidate.name,
                    voteCount: candidate.voteCount,
                    imageUrl: "https://api.dicebear.com/9.x/croodles/svg?seed=" + candidate.name.split(" ")[0],
                    isWinner: _winner
                }
                this.candidates.push(_candidate);
            }
        },
        async getVotingStatus() {
            this.votingEnabled = await this.contract.votingEnabled();
        },
        async getChairperson() {
            this.chairperson = await this.contract.chairperson();
            var _chairpersonAddress = ""
            _chairpersonAddress = this.chairperson
            this.chairpersonAddress = _chairpersonAddress.toUpperCase()
            var _senderAddress = this.accounts[0]
            this.senderAddress = _senderAddress.toUpperCase()
            this.isChairperson = (this.senderAddress == this.chairpersonAddress)
        },
        async getVoterHasVoted() {
            var voterInfo = await this.contract.voters(this.accounts[0]);
            this.hasVoted = voterInfo.voted;
        },
        async getWinner() {
            this.winner = await this.contract.getWinner();
        },
        disconnect(){
            this.accounts = []
            this.balance = 0
            this.provider = null
            this.contract = null
            this.candidates = []
            this.errorMessage = ""
            this.hasVoted = false
            this.votingEnabled = false
            this.chairperson = null
            this.candidatesLength = 0
            this.isChairperson = null;
        },
        async addVote(candidateIndex) {
            var signer = this.provider.getSigner();
            var contractWithSigner = this.contract.connect(signer)
            try{
                var transaction = await contractWithSigner.vote(candidateIndex)
                await transaction.wait()
                await this.getVoterHasVoted();
                await this.getWinner()
                this.getCandidates()
                this.errorMessage = ''
            } catch(error) {
                this.errorMessage = error.data.message
            }
        },
        async openVoting() {
            var signer = this.provider.getSigner();
            var contractWithSigner = this.contract.connect(signer)
            try{
                var transaction = await contractWithSigner.setVotingState(true)
                await transaction.wait()
                await this.getVotingStatus();
                this.errorMessage = ''
            } catch(error) {
                this.errorMessage = error.data.message
            }
        },
        async closeVoting() {
            var signer = this.provider.getSigner();
            var contractWithSigner = this.contract.connect(signer)
            try{
                var transaction = await contractWithSigner.setVotingState(false)
                await transaction.wait()
                await this.getVotingStatus();
                this.errorMessage = ''
            } catch(error) {
                this.errorMessage = error.data.message
            }
        },
        async getBalance() {
            var rawBalance = await this.provider.getBalance(this.accounts[0]);
            this.balance = utils.formatEther(rawBalance);
        },
        openLink(linkId, targetId) {
            var i, x, tablinks;
            x = document.getElementsByClassName("tab");
            for (i = 0; i < x.length; i++) {
                x[i].style.display = "none";
            }
            tablinks = document.getElementsByClassName("tablink");
                for (i = 0; i < x.length; i++) {
                tablinks[i].className = tablinks[i].className.replace(" w3-red", "");
            }
            document.getElementById(targetId).style.display = "block";
            document.getElementById(linkId).className += " w3-red";
        },
        confirmVote(index) {
            if(confirm("Is that your final answer?")) {
                this.addVote(index)
            }
        }
    }  
}
</script>
    
<style scoped>
    #contents {
        margin-left:130px;
    }
    #tabs {        
        min-height: 500px;
    }
</style>