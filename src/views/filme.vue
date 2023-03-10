<template>
    <div>
        <b-container fluid>
            <h2>
                {{ filme.nome }}
                <b-button v-if="getLoggedUser && getLoggedUser.role == 'admin'" id="removerFilme" @click='removerFilme'>
                    Remover</b-button>
            </h2>
            <span id="classificacaoFilme">{{ filme.classificacao }}</span>
            <div id="anexosFilme">
                <img :src="filme.imagem" id="imgFilme">
                <iframe width="560" height="315" :src="filme.trailer" title="YouTube video player" frameborder="0"
                    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
                    allowfullscreen id="trailer"></iframe>
            </div>
            <b-col cols="6">
                <div id="filmeAvaliacaoFavoritos">
                    <a id="rate"><span>{{ filme.avaliacao }}</span>
                        <b-icon icon="star-fill"></b-icon>/5.0
                    </a>
                    <b-button @click="avaliarModal()" id="avaliar" variant="info">Avaliar</b-button>
                    <b-button v-if="getLoggedUser !== null" @click="favoritos()" variant="info" id="favoritos"> &#9829;
                        Adicionar aos favoritos</b-button>
                    <b-button v-if="getLoggedUser !== null" @click="lista()" variant="info" id="lista">+ A minha lista
                    </b-button>
                </div>
                <div class="infoFilme">
                    <p><span>Sinopse: </span>{{ filme.sinopse }}</p>
                    <p><span>Ano: </span>{{ filme.ano }}</p>
                    <p><span>Realizador: </span><a class="infoFilme">{{ realizador }}</a></p>
                    <p><span>Produtora: </span><a class="infoFilme">{{ filme.produtora }}</a></p>
                    <p><span>Elenco: </span><a class="infoFilme">{{ elenco }} </a></p>
                    <p><span>Género: </span><a>{{ categoria }} </a></p>
                </div>
            </b-col>
            <b-row>
                <div id="zonaComentarios">
                    <hr>
                    <h3>Comentários</h3>
                    <div v-if="comentarios.length > 0">
                        <b-row class="comentario" v-for="(comentario, index) in comentarios" :key="index">
                            <b-col cols="1" class='imagemSpoiler'>
                                <img :src="utilizadoresFoto[index]" class="imagemUtilizador">
                                <input v-if="getLoggedUser !== null" @click='spoiler(comentario)' type="image"
                                    src="./image/spoiler.svg" class="botaoSpoiler">
                            </b-col>
                            <b-col class="coment">
                                <p id="username">{{ utilizadoresNome[index] }}</p>
                                <b-button v-if="comentario.spoiler == true" @click="comentario.spoiler = false"
                                    class="esconderSpoiler">VER SPOILER</b-button>
                                <p id="comentarioUser">{{ comentario.comentario }}</p>
                            </b-col>
                        </b-row>
                        <div id="comentarioUserFundo">
                        </div>
                    </div>
                    <p id="semComentarios" v-else>Ainda não tem comentarios. Seja o primeiro a comentar!</p>
                </div>
            </b-row>

            <b-modal ref='avaliarModal' id="avaliarModal" centered header-bg-variant="info" header-text-variant="light"
                body-bg-variant="light" footer-bg-variant="light">
                <template #modal-header="{ close }">
                    <h4 class='modalTitulo'>AVALIAR</h4>
                    <b-button @click="close" variant="info" class='fecharModal'>x</b-button>
                </template>
                <template>
                    <form @submit.prevent="avaliar">
                        <div>
                            <b-form-rating v-model="avaliacao"></b-form-rating>
                        </div>
                        <div>
                            <p>Comentário:</p>
                            <b-form-textarea id="textarea" v-model="comentario" placeholder="Seu comentario" rows="5">
                            </b-form-textarea>
                        </div>
                    </form>
                </template>
                <template #modal-footer>
                    <b-button id='avaliar' @click='avaliar()'>Avaliar</b-button>
                </template>
            </b-modal>

            <!-- Adicionado com sucesso -->
            <b-modal ref='adicionadoSucesso' id="adicionadoSucesso" header-bg-variant="info" header-text-variant="light"
                body-bg-variant="light" footer-bg-variant="light">
                <template #modal-header="{ close }">
                    <h4 class='modalTitulo'>Adicionado com sucesso</h4>
                    <b-button @click="close" variant="info" class='fecharModal'>x</b-button>
                </template>
                <template>
                    <p>O filme foi adicionado com sucesso a sua lista</p>
                </template>
                <template #modal-footer="{ close }">
                    <b-button @click='close()'>Fechar</b-button>
                </template>
            </b-modal>

        </b-container>

    </div>
</template>

<script>
import { mapGetters, mapMutations } from "vuex";

export default {
    name: 'PaginaFilme',
    data() {
        return {
            filme: null,
            avaliacao: '',
            comentario: '',
            loggedUtilizador: null,

            utilizadoresFoto: [],
            utilizadoresNome: [],
        }
    },

    computed: {
        ...mapGetters(['getFilmes', 'getFilme', 'getLoggedUser', 'getUtilizadores', 'getUtilizador', 'isFilmeFavoritoValido', 'isFilmeListaValido']),

        elenco() {
            return this.filme.elenco.join(', ');
        },

        categoria() {
            return this.filme.categoria.join(', ');
        },
        realizador() {
            return this.filme.realizador.join(', ');
        },

        comentarios() {
            for(const comentario of this.filme.comentarios){
                this.utilizadoresComentarios(comentario.utilizador)
            }
            return this.filme.comentarios;
        }
    },

    methods: {
        ...mapMutations(['SET_NOVO_COMENTARIO', 'SET_NOVA_AVALIACAO', 'SET_NOVO_FAVORITO', 'SET_NOVA_LISTA', 'SET_SPOILER', 'SET_REMOVER_FILME']),

        async getFilmeInfo() {
            try {
                await this.$store.dispatch("getFilme", this.$route.params.filmeID);
                this.filme = await this.getFilme.filme;

            } catch (error) {
                this.message =
                    (error.response && error.response.data) ||
                    error.message ||
                    error.toString();
            }
        },

        async avaliar() {
            if (this.avaliacao > 0) {
                await this.$store.dispatch("updateAvaliacao", [this.filme._id, this.avaliacao]);
                this.getFilmeInfo();
            }
            if (this.comentario.length > 0) {
                let novoComentario = {
                    utilizador: this.loggedUtilizador._id,
                    comentario: this.comentario,
                }
                await this.$store.dispatch("addComentario", [this.filme._id, novoComentario]);
                this.getFilmeInfo();
            }
            this.$refs['avaliarModal'].hide()
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

        async spoiler(comentario) {
            try {
                comentario.spoiler = comentario.spoiler == false ? true : false;
                await this.$store.dispatch("updateComentario", [this.filme._id, comentario.id, comentario.spoiler]);
            } catch (error) {
                this.message =
                    (error.response && error.response.data) ||
                    error.message ||
                    error.toString();
            }
        },

        async removerFilme() {
            if (confirm("Tens acerteza que queres remover este filme?")) {
                try {
                    await this.$store.dispatch("eliminarFilme", this.filme._id);
                    this.$router.push({ name: "filmes" });
                } catch (error) {
                    this.message =
                        (error.response && error.response.data) ||
                        error.message ||
                        error.toString();
                }
            }
        },

        avaliarModal() {
            if (this.getLoggedUser != '') {
                this.$refs['avaliarModal'].show()
            }
            else {
                alert('É preciso autenticar primeiro!')
            }
        },

        async favoritos() {
            if (this.isFilmeFavoritoValido(this.filme.nome)) {
                await this.$store.dispatch("addFavoritos", [this.loggedUtilizador._id, this.filme._id]);
                this.$refs['adicionadoSucesso'].show()
            }
        },

        async lista() {
            if (this.isFilmeListaValido(this.filme.nome)) {
                await this.$store.dispatch("addLista", [this.loggedUtilizador._id, this.filme._id]);
                this.$refs['adicionadoSucesso'].show()
            }
        },

        async utilizadoresComentarios(id) {
            try {
                await this.$store.dispatch("getUtilizador", id);
                let utilizador = await this.getUtilizador;
                this.utilizadoresFoto.push(utilizador.foto);
                this.utilizadoresNome.push(utilizador.nome);
            } catch (error) {
                this.message =
                    (error.response && error.response.data) ||
                    error.message ||
                    error.toString();
            }
        },
    },

    mounted() {
        this.getFilmeInfo();
        this.getLoggedUserInfo();
    },
}
</script>

<style scoped>
h2 {
    margin-top: 3%;
    margin-bottom: 1.5%;
}

#classificacaoFilme {
    font-family: var(--font1);
    font-size: 10pt;
}

#anexosFilme {
    height: 325px;
}

#imgFilme {
    position: relative;
    width: 10%;
    height: 65%;
    top: -13.5vh;
    z-index: 1;
}

#trailer {
    position: relative;
    left: -9.2vw;

}

#filmeAvaliacaoFavoritos {
    margin-bottom: 8vh;
}

button {
    margin-left: 1%;
    margin-right: 0.5%;
    margin-bottom: 0.5%;
}

span {
    font-family: var(--font2);
    font-size: 20px;
}

input {
    width: 100%;
    height: 20vh;
}

#rate {
    position: relative;
    top: -20%
}

#zonaComentarios {
    width: 100%;
    height: 100%;
}

.comentario {
    margin-top: 5vh;
}

.imagemUtilizador {
    width: 75px;
    height: 75px;
    border-radius: 50%;
}

#comentarioUser {
    font-family: var(--font1);
}

#comentarioUserFundo {
    width: 95%;
    height: 15%;
    background-color: var(--cor0)
}

#username {
    font-family: var(--font2);
    font-size: 18px;
    color: white;
}

.infoFilme {
    text-decoration: none;
    color: white;
    font-family: var(--font1);
}

.modalTitulo {
    margin: auto;
    font-family: var(--font2);
    color: white;
}

.fecharModal {
    position: absolute;
    right: 5px;
    top: 0px;
    color: white;
}

.imagemSpoiler {
    position: relative;
}

.botaoSpoiler {
    position: absolute;
    top: -10px;
    right: 25px;
    width: 30px;
}

.coment {
    position: relative;
}

.esconderSpoiler {
    position: absolute;
    left: -10px;
    top: 30px;
    width: 100%;
    height: 100%;
    font-family: var(--font1);
    background-color: var(--cor2)
}

#removerFilme {
    margin-left: 50px;
    background-color: var(--cor2);
    font-family: var(--font1);
    width: 100px;
}

#removerFilme:hover {
    opacity: 90%;
}

#semComentarios {
    font-family: var(--font1);
    text-align: center;
}
</style>