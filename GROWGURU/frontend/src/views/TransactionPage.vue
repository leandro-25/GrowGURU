<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Operações Financeiras</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <!-- Saldo Atual -->
      <ion-card>
        <ion-card-header>
          <ion-card-title>Saldo Atual</ion-card-title>
          <ion-card-subtitle>R$ {{ saldo.toFixed(2) }}</ion-card-subtitle>
        </ion-card-header>
      </ion-card>

      <!-- Formulário -->
      <ion-item>
        <ion-label position="stacked">Valor (R$)</ion-label>
        <ion-input v-model="valor" type="number" step="0.01"></ion-input>
      </ion-item>

      <ion-item>
        <ion-label>Tipo de Operação</ion-label>
        <ion-select v-model="tipo">
          <ion-select-option value="deposito">Depósito</ion-select-option>
          <ion-select-option value="saque">Saque</ion-select-option>
        </ion-select>
      </ion-item>

      <ion-item>
        <ion-label position="stacked">Descrição</ion-label>
        <ion-textarea v-model="descricao" rows="2"></ion-textarea>
      </ion-item>

      <ion-button 
        expand="block" 
        @click="executarTransacao"
        :disabled="!valor || valor <= 0"
      >
        Confirmar Operação
      </ion-button>

      <!-- Histórico -->
      <ion-list>
        <ion-list-header>
          <ion-label>Últimas Transações</ion-label>
        </ion-list-header>

        <ion-item v-for="transacao in historico" :key="transacao.id">
          <ion-label>
            <h3>{{ formatarData(transacao.data) }}</h3>
            <p>{{ transacao.descricao }}</p>
          </ion-label>
          <ion-note slot="end" :color="transacao.tipo === 'deposito' ? 'success' : 'danger'">
            R$ {{ transacao.valor.toFixed(2) }}
          </ion-note>
        </ion-item>
      </ion-list>

      <ion-button 
        expand="block" 
        fill="clear"
        @click="router.push('/tabs/estrategias')"
        class="ion-margin-top"
      >
        <ion-icon :icon="bookOutline" slot="start"></ion-icon>
        Ver Estratégias
      </ion-button>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { api } from '@/api';
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonItem,
  IonLabel,
  IonInput,
  IonButton,
  IonSelect,
  IonSelectOption,
  IonTextarea,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardSubtitle,
  IonList,
  IonListHeader,
  IonNote
} from '@ionic/vue';
import { bookOutline } from 'ionicons/icons';

const router = useRouter();
const valor = ref('');
const tipo = ref('deposito');
const descricao = ref('');
const saldo = ref(0);
const historico = ref([]);

const carregarDados = async () => {
  try {
    const token = localStorage.getItem('token');
    
    if (!token) {
      router.push('/login');
      return;
    }
    // Carregar saldo
    const { data: usuario } = await api.get('/usuarios', {
      headers: { Authorization: `Bearer ${token}` }
    });
    
    saldo.value = usuario.saldo;

    // Carregar histórico
    const { data: transacoes } = await api.get('/transacoes', {
      headers: { Authorization: `Bearer ${token}` }
    });
    
    historico.value = transacoes;

  } catch (error) {
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      router.push('/login');
    } else {
      console.error('Erro ao carregar dados:', error);
    }
  }
};

const executarTransacao = async () => {
  try {
    const token = localStorage.getItem('token');
    
    const { data } = await api.post('/transacoes', {
      valor: parseFloat(valor.value),
      tipo: tipo.value,
      descricao: descricao.value
    }, {
      headers: { Authorization: `Bearer ${token}` }
    });

    // Atualizar dados locais
    saldo.value = data.saldo_atual;
    historico.value.unshift({
      id: Date.now(),
      tipo: tipo.value,
      valor: parseFloat(valor.value),
      descricao: descricao.value,
      data: new Date().toISOString()
    });

    // Resetar formulário
    valor.value = '';
    descricao.value = '';

  } catch (error) {
    console.error('Erro na transação:', error);
    alert(error.response?.data?.error || 'Erro ao processar operação');
  }
};

const formatarData = (dataISO) => {
  return new Date(dataISO).toLocaleDateString('pt-BR', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  });
};

onMounted(() => {
  carregarDados();
});
</script>