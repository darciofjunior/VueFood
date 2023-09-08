<template>
    <div>
        <div class="row my-4">
            <h1 class="title-tenant">Detalhes do Pedido 
                <strong>{{ identify }}</strong>
            </h1>
        </div>

        <div class="row my-4">
        <div class="col-sm-12">
            <ul>
            <li>Número: {{ identify }}</li>
            <li>Total: {{ order.total | formatprice }}</li>
            <li>Data: {{ order.date | dateformat }}</li>
            <li>Status: 
                <span class="badge bg-success">{{ order.status }}</span>
            </li>
            <li v-if="order.table.identify">
                Mesa:
                <ul>
                <li>Identificador: {{ order.table.identify }}</li>
                <li>Descrição: {{ order.table.description }}</li>
                </ul>
            </li>
            <li v-if="order.client.name">
                Cliente:
                <ul>
                <li>Nome: {{ order.client.name }}</li>
                <li>E-mail: {{ order.client.email }}</li>
                </ul>
            </li>
            </ul>
        </div>
        </div> <!-- details order -->

        <hr>

        <div class="row my-4">
            <div class="col-12">
                <h4>Produtos</h4>
            </div>

            <div class="col-6 my-1" v-for="(product, index) in order.products" :key="index">
                <div class="restaurant-card">
                    <a href="#">
                        <img class="card-img-top" :src="product.image" :alt="product.title"/>
                    </a>
                    <div class="restaurant-card-body">
                        <h5>{{ product.title }}</h5>
                    </div>
                </div>
            </div>
        </div>
        <!-- products order -->

        <hr>
        <button class="btn btn-success"
            @v-if="me.name != '' && me.name === order.client.name && order.evaluations.length === 0"
            @click.prevent="openModalEvaluation">
            Avaliar o Pedido
        </button>

        <modal name="evaluation-order" :height="350">
            <div class="px-md-5 my-4">
                <h1>Avaliar o pedido {{ identify }}</h1>

                <strong>Nota: </strong>

                <star-rating v-bind:increment="0.5"
                    v-bind:max-rating="3"
                    inactive-color="#000"
                    active-color="#f00"
                    v-bind:star-size="90"
                    v-model="evaluation.stars"/>

                <div class="form-group">
                    <textarea name="comment" v-model="evaluation.comment"  cols="30" rows="3" placeholder="Comentário (Opcional)" class="form-control"></textarea>
                </div>
                <button class="btn btn-info" 
                    @click.prevent="sendEvaluation"
                    :disabled="loadSendEvaluation">
                    <span v-if="loadSendEvaluation">Enviando ...</span>
                    <span v-else>Avaliar</span>
                </button>
            </div>
        </modal>
        
        <div class="evaluations-order col-12" v-if="order.evaluations.length">
            <div v-for="(evaluation, index) in order.evaluations" :key="index">
                <p><strong>Nome: </strong>{{ evaluation.client.name }}</p>
                <p><strong>Comentário: </strong>{{ evaluation.comment }}</p>
                <p><strong>Nota: </strong>
                    <star-rating 
                        v-bind:increment="0.5"
                        v-bind:max-rating="3"
                        inactive-color="#000"
                        active-color="#f00"
                        v-bind:star-size="90"
                        :rating="evaluation.stars">
                    </star-rating>
                </p>
            </div>
        </div>
        <!-- Evaluations -->

    </div>
</template>

<script>
import { mapActions, mapState } from 'vuex'

export default {
    props: ['identify'],

    computed: {
        ...mapState({
            me: state => state.auth.me
        }),
    },
    
    created() {
        this.getOrderByIdentify(this.identify)
                .then(response => this.order = Object.assign(response.data.data))
                .catch(error => {
                    this.$toast.error('Falha ao carregar detalhes do Pedido !!!')
                    this.$router.push({name: 'home'})
                })
    },

    data() {
        return {
            order: {
                identify: '',
                date: '',
                status: '',
                total: '',
                table: {
                    identify: '',
                    description: '',
                },
                client: {
                    name: '',
                    email: ''
                },
                company: {
                    name: '',
                    image: ''
                },
                products: [],
                evaluations: []
            },
            evaluation: {
                stars: 1,
                comment: ''
            },
            loadSendEvaluation: false,
        }
    },

    methods: {
        ...mapActions([
            'getOrderByIdentify',
            'evaluationOrder'
        ]),

        openModalEvaluation () {
            this.$modal.show('evaluation-order')
        },

        closeModalEvaluation () {
            this.$modal.hide('evaluation-order')
        },

        sendEvaluation () {
            this.loadSendEvaluation = true

            const params = {
                identify: this.identify,
                ...this.evaluation
            }

            this.evaluationOrder(params)
                .then(response => {
                    this.$toast.success('Avaliação enviada com sucesso !!!')
                    this.order.evaluations.push(response.data.data)
                    this.closeModalEvaluation()
                })
                .catch(error => this.$toast.error('Falha ao enviar a avaliação !!!'))
                .finally(() => this.loadSendEvaluation = false)
        }
    }
}
</script>