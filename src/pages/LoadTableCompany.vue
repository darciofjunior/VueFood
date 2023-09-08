<template>
    <div class="my-4 text-center">
      <h1 class="title-tenant">Carregando dados, aguarde!</h1>
    </div>
</template>

<script>
import { mapActions, mapState } from 'vuex'

export default {
  props: ['token_company', 'token_table'],

  computed: {
    ...mapState({
      company: state => state.companies.companySelected,
    }),
  },

  created() {
    this.getCompanyByToken(this.token_company)
        .then(response => {
          const params = {table: this.token_table, token_company: this.token_company}

          this.getTableFromCompany(params)
              .then(response => this.$toast.success('Loja carregada com Sucesso'))
              .catch(response => this.$toast.error('Falha ao carregar a Empresa.'))
              .finally(() => this.$router.push({name: 'products', params: {companyFlag: this.flag}}))
        })
        .catch(response => {
          this.$toast.error('Falha ao carregar a Empresa.')

          this.$router.push({name: 'home'})
        })
  },

  methods: {
    ...mapActions([
      'getCompanyByToken',
      'getTableFromCompany'
    ]),
  },
}
</script>