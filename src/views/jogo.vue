<template>
    <div>
        <b-container>
            <h1>{{ jogo.nome }}</h1>
            <div v-if="jogo.tipo == 'Quizz'">
                <div class="jogo quizz" v-for="(jogo, index) in jogo.perguntas" :key="index">
                    <b-row align-h="center">
                        <b-col cols="10" class="pergunta">
                            <p>{{ jogo.pergunta }}</p>
                        </b-col>
                    </b-row>
                    <b-row align-h="around">
                        <b-col class="alternativas" cols='4' v-for="(alternativa, ind) in jogo.alternativas" :key="ind">
                            <b-button variant="outline-dark" class='botao botaoJogo'
                                :class="respostasUtilizador[index] == alternativa ? 'selecionada' : 'naoSelecionada'"
                                @click='selecionar(alternativa, index)'>{{ alternativa }}</b-button>
                        </b-col>
                    </b-row>
                </div>

            </div>
            <div v-if="jogo.tipo == 'Preencher'" id='preencher'>
                <div class="jogo" v-for="(jogo, index) in jogo.perguntas" :key="index">
                    <h2>{{ jogo.pergunta }}</h2>
                    <div>
                        <img v-if="jogo.tipoAnexo == 'Imagem'" :src="jogo.anexo" height="350" class="anexoJogo">
                        <iframe v-else width="560" height="315" :src="jogo.anexo" title="YouTube video player"
                            frameborder="0"
                            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
                            allowfullscreen class="anexoJogo"></iframe>
                    </div>
                    <div>
                        <b-form-input class="respostas" v-model="respostasUtilizador[index]" placeholder="Sua resposta"
                            required></b-form-input>
                    </div>

                </div>
            </div>
            <div v-if="jogo.tipo == 'Lista'">
                <div class="jogo">
                    <h2>{{ jogo.perguntas[0].pergunta }}</h2>
                    <div>
                        <img v-if="jogo.perguntas[0].tipoAnexo == 'Imagem'" :src="jogo.perguntas[0].anexo" height="350"
                            class="anexoJogo">
                        <iframe v-else width="560" height="315" :src="jogo.perguntas[0].anexo"
                            title="YouTube video player" frameborder="0"
                            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
                            allowfullscreen class="anexoJogo"></iframe>
                    </div>
                    <div>
                        <b-form-input class="respostas" v-model="respostasUtilizadorLista" placeholder="Sua resposta"
                            @change="verificarRespostaLista()"></b-form-input>
                    </div>
                    <div>
                        <b-row align-h="center" id="jogoListaTabela">
                            <b-col cols="5" class="respostaLista"
                                v-for="(resposta, index) in jogo.perguntas[0].resposta" :key="index">
                                <span v-if="resposta == respostasUtilizador[index]">{{ resposta }}</span>
                            </b-col>
                        </b-row>
                    </div>
                </div>
            </div>

            <b-row>
                <b-button class="botao" id="terminarQuizz" @click="terminar(jogo._id)">TERMINAR QUIZZ</b-button>
            </b-row>
        </b-container>
    </div>
</template>

<script>
import { mapGetters, mapMutations } from "vuex";

export default {
    name: 'PaginaJogo',
    data() {
        return {
            jogo: null,
            respostasUtilizador: [],
            respostasUtilizadorLista: '',
            certas: 0,

            loggedUtilizador: null,
        }
    },

    computed: {
        ...mapGetters(['getLoggedUser', 'getJogos', 'getJogo', 'getDesafios', 'getUtilizador']),
    },

    methods: {
        ...mapMutations(['SET_NOVA_CLASSIFICACAO', 'SET_DESAFIO']),

        async getJogoInfo() {
            try {
                await this.$store.dispatch("getJogo", this.$route.params.jogoID);
                this.jogo = await this.getJogo.jogo;

            } catch (error) {
                this.message =
                    (error.response && error.response.data) ||
                    error.message ||
                    error.toString();
            }
        },

        async terminar(id) {
            let quantPerguntas = 0;
            if (this.jogo.tipo != 'Lista') {
                for (let i = 0; i < this.jogo.perguntas.length; i++) {
                    if (this.jogo.perguntas[i].resposta == this.respostasUtilizador[i]) {
                        this.certas++
                    }
                    quantPerguntas++;
                }
            }
            else {
                for (let i = 0; i < this.jogo.perguntas[0].resposta.length; i++) {
                    if (this.jogo.perguntas[0].resposta[i] == this.respostasUtilizador[i]) {
                        this.certas++
                    }
                    quantPerguntas++;
                }
            }

            const pontuacao = this.certas * 25;

            const novaClassificacao = {
                utilizadorID: this.getLoggedUser.id,
                pontuacao: pontuacao,
            }

            try {
                await this.$store.dispatch("addClassificacao", [this.jogo._id, novaClassificacao]);
                await this.$store.dispatch("addDesafio", [this.getLoggedUser.id, this.jogo._id]);
                await this.$store.dispatch("getAllDesafios");

                await this.$store.dispatch("addDesafioConcluido", [this.loggedUtilizador, this.getDesafios]);
                this.$router.push({ name: "classificacao", params: { jogoID: id, certas: this.certas, numPerguntas: quantPerguntas } });
            } catch (error) {
                this.message =
                    (error.response && error.response.data) ||
                    error.message || error.toString();
            }
        },

        async getLoggedUserInfo() {
            try {
                let utilizador = await this.getLoggedUser
                await this.$store.dispatch("getUtilizador", utilizador.id);
                this.loggedUtilizador = await this.getUtilizador;
            } catch (error) {
                this.message =
                    (error.response && error.response.data) ||
                    error.message ||
                    error.toString();
            }
        },


        selecionar(alternativa, index) {
            this.respostasUtilizador[index] = alternativa
            this.$forceUpdate()
        },

        verificarRespostaLista() {
            for (let i = 0; i < this.jogo.perguntas[0].resposta.length; i++) {
                if (this.jogo.perguntas[0].resposta[i] == this.respostasUtilizadorLista) {
                    this.respostasUtilizador[i] = this.respostasUtilizadorLista
                    this.$forceUpdate()
                }
            }
        }
    },

    mounted() {
        this.getJogoInfo();
        this.getLoggedUserInfo()
    },
}
</script>

<style scoped>
h1 {
    margin-top: 5vh;
    margin-bottom: 5vh;
    text-align: center;
    font-family: var(--font2);
    font-size: 2.5em;
}

#preencher {
    margin: auto;
}

#preencher>.jogo {
    width: 75%;
    margin: auto;
    margin-bottom: 15vh;
    text-align: center;
}

.anexoJogo {
    margin: 5vh;
}

.respostas {
    margin: auto;
    width: 35%;
}

.jogo {
    margin-bottom: 7.5vh;
    text-align: center;
}

.botao {
    border-color: var(--cor1);
    font-family: var(--font1);
    font-size: 1.25em;
    color: black;
}

.pergunta {
    margin: 3vh;
    height: 100px;
    background-color: var(--cor4);
    text-align: center;
}

.pergunta>p {
    font-family: var(--font1);
    font-size: 1.5em;
    color: black;
    padding-top: 2.5%;
}

.alternativas {
    margin: 2vh;
    height: 100px;
    text-align: center;
}

.alternativas>.botao {
    height: 90%;
    width: 100%;
}

.alternativas>.naoSelecionada {
    background-color: var(--cor4);
}

.alternativas>.naoSelecionada:hover {
    background-color: var(--cor3);
}

.alternativas>.selecionada {
    background-color: var(--cor3);
}

.alternativas>button:active {
    background-color: var(--cor3);
    box-shadow: inset 5px 5px 10px 0px rgba(0, 0, 0, 0.575);
}

#jogoListaTabela {
    width: 50vw;
    margin: auto;
    margin-top: 10vh;
}

.respostaLista {
    height: 45px;
    text-align: center;
    padding-top: 10px;
    background-color: var(--cor3);
    font-family: var(--font1);
    font-size: 1.1em;
    color: black;
}

.respostaLista:nth-of-type(2n+1) {
    background-color: var(--cor4);
}

#terminarQuizz {
    margin: auto;
    margin-top: 5vh;
    background-color: var(--cor3);
    height: 100px;
    width: 25vw;
}

#terminarQuizz:active {
    box-shadow: inset 5px 5px 10px 0px rgba(0, 0, 0, 0.575);
}
</style>