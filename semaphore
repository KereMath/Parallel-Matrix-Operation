#include <iostream>
#include <vector>
#include <pthread.h>
#include <limits.h>
#include "hw2_output.h"
#include <semaphore.h>


using Matrix = std::vector<int>;
using namespace std;

struct AdditionData1 {
     int row;
     int M;
     Matrix* Jmat;
     Matrix* Lmat;
     Matrix* Rmat;
     vector<sem_t>* semaphores2;
};

struct AdditionData2 {
     int row;
     int K;
     Matrix* Jmat;
     Matrix* Lmat;
     Matrix* Rmat;
     vector<sem_t>* semaphores1;
};

struct MultiplicationData {
     int row;
     int M;
     int K;
     Matrix* Jmat;
     Matrix* Lmat;
     Matrix* Rmat;
     vector<sem_t>* semaphores2;
     vector<sem_t>* semaphores1;
};
int counter;
vector<sem_t> semaphores2;
vector<sem_t> semaphores1;
vector<int>flags;
void* addition1(void* arg);
void* addition2(void* arg);
void* multiplication(void* arg);
vector<pthread_mutex_t> flags_mutex;

int main() {
     hw2_init_output();
     int N, M, K;
     cin >> N >> M;
     Matrix A(N * M);
     for (int i = 0; i < N; ++i) {
         for (int j = 0; j < M; ++j) {
             cin >> A[i * M + j];
         }
     }

     cin >> N >> M;
     Matrix B(N * M);
     for (int i = 0; i < N; ++i) {
         for (int j = 0; j < M; ++j) {
             cin >> B[i * M + j];
         }
     }

     cin >> M >> K;
     Matrix C(M * K);
     for (int i = 0; i < M; ++i) {
         for (int j = 0; j < K; ++j) {
             cin >> C[i * K + j];
         }
     }

     cin >> M >> K;
     Matrix D(M * K);
     for (int i = 0; i < M; ++i) {
         for (int j = 0; j < K; ++j) {
             cin >> D[i * K + j];
         }
     }
     counter=M;
flags.resize(K);
for(int i=0;i<K;i++){
    flags[i]=0;
}
flags_mutex.resize(K);
for (int i = 0; i < K; ++i) {
    pthread_mutex_init(&flags_mutex[i], 0);
}

Matrix J(N * M, 0);
Matrix L(M * K, 0);
Matrix R(N * K, 0);

semaphores2.resize(N);
for (int i = 0; i < N; ++i) {
     sem_init(&semaphores2[i], 0, 0);
}

semaphores1.resize(K);
for (int i = 0; i < K; ++i) {
     sem_init(&semaphores1[i], 0, 0);
}

     vector<pthread_t> ADD1(N);
     vector<pthread_t> ADD2(M);
     for (int i = 0; i < N; ++i) {
         AdditionData1* add_data = new AdditionData1{i, M, &A, &B, &J,
&semaphores2};
         pthread_create(&ADD1[i], 0, addition1,
add_data);
     }

     for (int i = 0; i < M; ++i) {
         AdditionData2* add_data = new AdditionData2{i, K, &C, &D, &L,
&semaphores1};
         pthread_create(&ADD2[i], 0, addition2,
add_data);
     }

     vector<pthread_t> MULT(N);
     for (int i = 0; i < N; ++i) {
         MultiplicationData* mult_data = new MultiplicationData{i, M, K,
&J, &L, &R, &semaphores2, &semaphores1};
         pthread_create(&MULT[i], 0,
multiplication, mult_data);
     }

     for (int i = 0; i < N; ++i) {
         pthread_join(ADD1[i], 0);
     }

     for (int i = 0; i < M; ++i) {
         pthread_join(ADD2[i], 0);
     }
     for (int i = 0; i < N; ++i) {
         pthread_join(MULT[i], 0);
     }
for (int i = 0; i < N; ++i) {
     sem_destroy(&semaphores2[i]);
}

for (int i = 0; i < K; ++i) {
     sem_destroy(&semaphores1[i]);
}

     for (int i = 0; i < N; ++i) {
         for (int j = 0; j < K; ++j) {
            if(j==K-1){cout << R[K*i+j];}
            else{
             cout << R[K*i+j] << " ";}
         }
         cout << endl;
     }




     return 0;
}

void* addition1(void* arg) {
     AdditionData1* data = static_cast<AdditionData1*>(arg);

     for (int j = 0; j < data->M; ++j) {
         (*data->Rmat)[data->row * data->M + j] = (*data->Jmat)[data->row * data->M + j] +
(*data->Lmat)[data->row * data->M + j];
         hw2_write_output(0, data->row+1, j+1,
(*data->Rmat)[data->row * data->M + j]);
     }

     sem_post(&(*data->semaphores2)[data->row]);

     delete data;
     return 0;
}



void* addition2(void* arg) {
     AdditionData2* data = static_cast<AdditionData2*>(arg);

     for (int k = 0; k < data->K; ++k) {
    (*data->Rmat)[data->row * data->K + k] = (*data->Jmat)[data->row * data->K + k] +
    (*data->Lmat)[data->row * data->K + k];
    hw2_write_output(1, data->row+1, k+1, (*data->Rmat)[data->row * data->K + k]);

    pthread_mutex_lock(&flags_mutex[k]);
    flags[k]++;
    pthread_mutex_unlock(&flags_mutex[k]);
    if(flags[k]==counter){
        sem_post(&(*data->semaphores1)[k]);}
}





     delete data;
     return 0;
}

void* multiplication(void* arg) {
     MultiplicationData* data = static_cast<MultiplicationData*>(arg);
         sem_wait(&(*data->semaphores2)[data->row]);

     for (int k = 0; k < data->K; ++k) {
         sem_wait(&(*data->semaphores1)[k]);
         int sum = 0;
         for (int m = 0; m < data->M; ++m) {
             int val1 = (*data->Jmat)[data->row * data->M + m];
             int val2 = (*data->Lmat)[m * data->K + k];

             sum += val1 * val2;
         }
         sem_post(&(*data->semaphores1)[k]);

         (*data->Rmat)[data->row * data->K + k] = sum;
         hw2_write_output(2, data->row+1, k+1, (*data->Rmat)[data->row * data->K + k]);
     }
         sem_post(&(*data->semaphores2)[data->row]);

     delete data;
     return 0;
}
