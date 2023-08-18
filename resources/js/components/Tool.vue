<template>
  <loading-view :loading="loading">
  <heading class="mb-6">Autenticação de dois fatores (Google 2FA)</heading>
    <loading-card :loading="loading" class="card" style="max-width: 600px">
      <div class="p-3" v-if="status.confirmed == 1">
        <p class="mb-4">
          Autenticação de dois fatores está ativa.
        </p>
      </div>

      <div v-else class="p-3">
        <p class="p-2">A autenticação de dois fatores (2FA) fortalece a segurança de acesso exigindo dois métodos (também conhecidos como fatores) para verificar sua identidade. A autenticação de dois fatores protege contra phishing, engenharia social e ataques de força bruta de senha e protege seus logins de invasores que exploram credenciais fracas ou roubadas.</p>

        <h3 class="p-3 my-4">Código de recuperação</h3>

        <p class="p-2 no-print">
          <strong>
            1) Baixe o aplicativo Google Authenticator no seu celular. 
          </strong>
        </p>

        <div class="p-3">
          <p>
            <strong>2) Digitalize este código QR usando o aplicativo Authenticator do Google para configurar e insira o código de 6 dígitos.</strong>
          </p>
          <div class="text-center">
            <img width="150" :src="twofa.google2fa_url" alt="qr_code">
          </div>
          <br>
          <input v-model="form.otp" @keyup="autoSubmit()" placeholder="Insira o código de 6 dígitos aqui" type="text"
                 class="w-full form-control form-input form-input-bordered mb-2">
          <button @click="confirmOtp" class="btn btn-default btn-primary">Ativar 2FA</button>
        </div>

      </div>

    </loading-card>
  </loading-view>
</template>

<script>
export default {
  data() {
    return {
      twofa: [],
      form: {},
      status: null,
      loading: true
    }
  },

  mounted() {
    this.getStatus()
    this.getRecoveryCodes()
  },

  methods: {
    getStatus() {
      Nova.request().get('/nova-vendor/nova-two-factor/status')
          .then(res => {
            this.status = res.data
            this.loading = false
          })
    },

    getRecoveryCodes() {
      Nova.request().get('/nova-vendor/nova-two-factor/register')
          .then(res => {
            this.twofa = res.data
          })
    },

    toggle() {
      Nova.request().post('/nova-vendor/nova-two-factor/toggle', {
        status: this.status.enabled
      })
          .then(res => {
            this.$toasted.show(res.data.message, {type: 'success'})
          })
    },

    confirmOtp() {
      Nova.request().post('/nova-vendor/nova-two-factor/confirm', this.form)
          .then(res => {
            this.$toasted.show(res.data.message, {type: 'success'})
            this.getStatus()
          })
          .catch(err => {
            this.$toasted.show(err.response.data.message, {type: 'error'})
          })
    },

    autoSubmit() {
      if (this.form.otp.length === 6) {
        this.confirmOtp()
      }
    },
    downloadAsText(filename, text) {
      var element = document.createElement('a');
      element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(text));
      element.setAttribute('download', filename);

      element.style.display = 'none';
      document.body.appendChild(element);

      element.click();

      document.body.removeChild(element);
    }
  }
}
</script>
