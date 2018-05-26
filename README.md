Escalonamento-em-C-
Estudo de um simulador de escalonamento utilizando a linguagem C. 


    /* Simulador de Escalonamento de Processos*/
    #include <stdio.h>
    #include <stdlib.h>
    /* Estrutura */
    struct processos {
        int id;                     /* Identifição do processo*/              
     int surto;                     /* Tempo de duração do processo*/
        int prioridade;           
        int execucao;               /* Tempo de execução do processo*/
        int espera;                 /* Tempo de espera do processo*/
        struct processos *prox;
    };
    /* Declarações de Protótipo de função */
    struct processos *init_processos (int id, int surto, int prioridade);
    void fcfs (struct processos *proc);
    void listprocs (struct processos *proc);
    void prioridade (struct processos *proc);
    void rr (struct processos *proc, int quantum);
    void sjf (struct processos *proc);

    int main (void) {

        struct processos *plist, *ptmp;
        plist       = init_processos(1, 10, 3);
        plist->prox = init_processos(2,  1, 1); ptmp = plist->prox;
        ptmp->prox  = init_processos(3,  2, 3); ptmp = ptmp->prox;
        ptmp->prox  = init_processos(4,  1, 4); ptmp = ptmp->prox;
        ptmp->prox  = init_processos(5,  5, 2);
        /* Simulações executadas*/
        listprocs(plist);
        fcfs(plist);
        sjf(plist);     
        prioridade(plist);
        rr(plist,0);

        while (plist != NULL) {
            ptmp = plist;
            plist = plist->prox;
            free(ptmp);
        };
        return(0);
    };
    /* Inicialização de entrada da lista de processos*/
    struct processos *init_processos (int id, int surto, int prioridade) {
        struct processos *proc;
     proc = (struct processos*)malloc(sizeof(struct processos)); 
      if (proc == NULL) {
            printf("Erro Fatal: Falha Alocacao de memoria.\nFinalizar.\n");
            exit(1);
        };
        proc->id = id;
        proc->surto = surto;
        proc->prioridade = prioridade;
        proc->execucao = 0;
        proc->espera = 0;
        proc->prox = NULL;
        return(proc);
    };

    /* Escalonamento o primeiro que chega 
    /* é o primeiro a sair, ou seja, será executado primeiro */
    void fcfs (struct processos *proc) {
        int tempo = 0, inicio, fim;
      struct processos *tmp = proc;
      printf("\tEscalonamento FCFS\n");
        printf("\n");
      while (tmp != NULL) {
        inicio = tempo;
        tempo += tmp->surto;
        fim = tempo;
        printf("Processo: %d\tSurto: %d\tEspera: %d\tRetorno: %d\n", tmp->id, tempo, inicio, fim);
        tmp = tmp->prox;
      };
        printf("\n\n");
    };

    /* Listando Processos */
    void listprocs (struct processos *proc) {
      struct processos *tmp = proc;
      printf("\tListagem de Processos\n");
      printf("\n");
      while (tmp != NULL) {
        printf("Processo: %d\tPrioridade: %d\tSurto: %d\n", tmp->id, tmp->prioridade, tmp->surto);
        tmp = tmp->prox;
      };
      printf("\n\n");
     };
    /* Simulação de Processos por Prioridade
     * Obs: O processo de menor valor de prioridade obtem
     * prioridade maior na fila de processos */
    void prioridade (struct processos *proc) {
      int tempo, inicio, fim, maior;
      struct processos *copia, *tmpsrc, *tmp, *maiorprimeiro;
      printf("\tEscalonamento por Prioridade\n");
       printf("\n");

         /* Replicando Lista de Processos */
      tmpsrc = proc;
      copia = tmp = NULL;
      while (tmpsrc != NULL) {
        if (copia == NULL) {
        copia = init_processos(tmpsrc->id, tmpsrc->surto, tmpsrc->prioridade);
        tmp = copia;
        } else {
        tmp->prox = init_processos(tmpsrc->id, tmpsrc->surto, tmpsrc->prioridade);
        tmp = tmp->prox;
        };
        tmpsrc = tmpsrc->prox;
      };
      /* Programa Principal */
      tempo = 0;
      while (copia != NULL) {

              /* Localiza o proximo processo */
        maiorprimeiro = NULL;
        maior = copia->prioridade;
        tmp = copia->prox;
        tmpsrc = copia;
        while (tmp != NULL) {
        if (tmp->prioridade < maior) {
          maior = tmp->prioridade;
          maiorprimeiro = tmpsrc;
        };
        tmpsrc = tmp;
        tmp = tmp->prox;
        };
         if (maiorprimeiro == NULL) {
        /* Verifica se o primeiro processo possui maior prioridade */
        inicio = tempo;
        tempo += copia->surto;
        fim = tempo;
        printf("Processo: %d\tSurto: %d\tEspera: %d\tRetorno: %d\n", copia->id, tempo, inicio, fim);
        tmpsrc = copia->prox;
        free(copia);
        copia = tmpsrc;
        } else {
        /* Verifica se o primeiro processo não possui maior prioridade */
        tmp = maiorprimeiro->prox;
        inicio = tempo;
        tempo += tmp->surto;
        fim = tempo;
        printf("Processo: %d\tSurto: %d\tEspera: %d\tRetorno: %d\n", tmp->id, tempo, inicio, fim);
        maiorprimeiro->prox = tmp->prox;
        free(tmp);
        };
      };
      printf("\n\n");
    };
    /* Escalonamento Round-Robin */
    void rr (struct processos *proc, int quantum) {
      int jobsremain, passes;
      struct processos *copia, *tmpsrc, *tmp, *slot;
      printf("\tEscalonamento Round-Robin - Quantum: %d)\n", quantum);
      printf("\n");
      tmpsrc = proc;
      copia = tmp = NULL;
      while (tmpsrc != NULL) {
        if (copia == NULL) {
        copia = init_processos(tmpsrc->id, tmpsrc->surto, tmpsrc->prioridade);
        tmp = copia;
        } else {
        tmp->prox = init_processos(tmpsrc->id, tmpsrc->surto, tmpsrc->prioridade);
        tmp = tmp->prox;
        };
        tmpsrc = tmpsrc->prox;
      };
      /* Programa rotina de análise de prioridade  */
      jobsremain = 2;
      slot = NULL;
      while (jobsremain) {
        jobsremain = 0;
        /* Seleciona o próximo processo efetuando sua alocação */
        if (slot == NULL) {
        slot = copia;
        jobsremain = 2;
        } else {
        passes = 0;
        do {
          if (slot->prox == NULL) {
            passes++;
            slot = copia;
          } else {
            slot = slot->prox;
          };
        } while (passes <= 3 && slot->surto == slot->execucao);
        if (passes <= 3) {
          jobsremain = 2;
        };
        };
        /* Executa um ciclo */
        tmp = copia;
        while (tmp != NULL) {
        if (tmp->surto > tmp->execucao) {
          if (tmp == slot) {
            tmp->execucao += quantum;
          } else {
            tmp->espera += quantum;
          };
        };
        tmp = tmp->prox;
        };
      };
      /* Exibe os resultados e elimina as replicações */
      tmp = copia;
      while (tmp != NULL) {
        printf("Processo: %d\tSurto: %d\tEspera: %d\tRetorno: %d\n", tmp->id, tmp->surto, tmp->espera, tmp->execucao + tmp->espera);
        tmpsrc = tmp;
        tmp = tmp->prox;
        free(tmpsrc);
      };
      printf("\n");
    };

    /* Escalonamento SJF*/
    void sjf (struct processos *proc) {
      int tempo, inicio, fim, shortest;
      struct processos *copia, *tmpsrc, *tmp, *beforeshortest;
      printf("\tEscalonamento SJF\n");
      printf("\n");
      /* Lista de processos é replicada */
      tmpsrc = proc;
      copia = tmp = NULL;
      while (tmpsrc != NULL) {
        if (copia == NULL) {
        copia = init_processos(tmpsrc->id, tmpsrc->surto, tmpsrc->prioridade);
        tmp = copia;
        } else {
        tmp->prox = init_processos(tmpsrc->id, tmpsrc->surto, tmpsrc->prioridade);
        tmp = tmp->prox;
        };
        tmpsrc = tmpsrc->prox;
      };
      tempo = 0;
      while (copia != NULL) {
        /* Encontra o proximo processo*/
        beforeshortest = NULL;
        shortest = copia->surto;
        tmp = copia->prox;
        tmpsrc = copia;
        while (tmp != NULL) {
        if (tmp->surto < shortest) {
          shortest = tmp->surto;
          beforeshortest = tmpsrc;
        };
        tmpsrc = tmp;
        tmp = tmp->prox;
        };
        /* Executa processo e remove ráplica da lista de processos */
        if (beforeshortest == NULL) {
        /* Aloca o primeiro processo caso o mesmo seja menor */
        inicio = tempo;
        tempo += copia->surto;
        fim = tempo;
        printf("Processo: %d\tSurto: %d\tEspera: %d\tRetorno: %d\n", copia->id, tempo, inicio, fim);
        tmpsrc = copia;
        copia = copia->prox;
        free(tmpsrc);
        } else {
        /* Aloca o primeiro processo caso não haja 
        ocorrencia de outro menor 
        */
        tmp = beforeshortest->prox;
        inicio = tempo;
        tempo += tmp->surto;
        fim = tempo;
        printf("Processo: %d\tSurto: %d\tEspera: %d\tRetorno: %d\n", tmp->id, tempo, inicio, fim);
        beforeshortest->prox = tmp->prox;
        free(tmp);
        }
      }
         printf("\n\n\n\tAperte qualquer tecla para execultar o escalonamento RR\n\n\n");
         printf("\n");
         system ("pause");

    }
