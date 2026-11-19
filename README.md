### Mem bandwidth
```
mkdir -p ~/bench && cat > ~/bench/rdbw.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include <omp.h>

#define N 500000000UL   /* 4 GB, un solo array */
#define NTIMES 5

int main(void) {
    double *a = aligned_alloc(64, N*sizeof(double));
    if (!a) { fprintf(stderr, "allocazione fallita\n"); return 1; }

    #pragma omp parallel for
    for (size_t i = 0; i < N; i++) a[i] = 1.0;

    double best = 1e30, keep = 0.0;
    for (int k = 0; k < NTIMES; k++) {
        double sum = 0.0;
        double t = omp_get_wtime();
        #pragma omp parallel for reduction(+:sum)
        for (size_t i = 0; i < N; i++) sum += a[i];
        t = omp_get_wtime() - t;
        if (t < best) best = t;
        keep += sum;
    }

    printf("%2d thread -> Read: %6.1f GB/s  (checksum %.0f)\n",
           omp_get_max_threads(), 8.0*N/best/1e9, keep);
    free(a);
    return 0;
}
EOF
gcc -O3 -march=native -fopenmp -o ~/bench/rdbw ~/bench/rdbw.c || exit 1
for t in 8 16 32; do OMP_NUM_THREADS=$t ~/bench/rdbw; done
```

### Mem stress
```
sudo apt install -y stressapptest
sudo stressapptest -M 96000 -s 900 -W
```
