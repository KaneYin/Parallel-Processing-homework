#include "mpi.h"
#include <math.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[])
{
    int n, myid, numprocs, i, p1=0, p2=0;
    double PI25DT = 3.141592653589793238462643;
    double mypi, pi, h, sum, x;
    double t_start, t_end, t_total, t_start_comp, t_comp;

    MPI_Init(&argc, &argv);
    MPI_Comm_size(MPI_COMM_WORLD, &numprocs);
    MPI_Comm_rank(MPI_COMM_WORLD, &myid);

    if (myid == 0) {
        if (argc < 4) { printf("Usage: ./par_pi <n> <p1> <p2> [outfile]\n"); n=0; }
        else { n=atoi(argv[1]); p1=atoi(argv[2]); p2=atoi(argv[3]); }
    }

    MPI_Barrier(MPI_COMM_WORLD);
    t_start = MPI_Wtime();

    MPI_Bcast(&n, 1, MPI_INT, 0, MPI_COMM_WORLD);

    MPI_Barrier(MPI_COMM_WORLD);
    t_start_comp = MPI_Wtime();

    if (n > 0) {
        h = 1.0/(double)n;
        sum = 0.0;
        for (i = myid+1; i <= n; i += numprocs) {
            x = h*((double)i - 0.5);
            sum += 4.0/(1.0+x*x);
        }
        mypi = h * sum;

        /* Hypercube recursive reduction using Send/Recv */
        {
            int k, partner, active=1, num_rounds=0, temp=numprocs;
            double recv_val;
            MPI_Status status;

            while (temp > 1) { num_rounds++; temp >>= 1; }

            for (k = 0; k < num_rounds && active; k++) {
                partner = myid ^ (1 << k);
                if (myid & (1 << k)) {
                    MPI_Send(&mypi, 1, MPI_DOUBLE, partner, k, MPI_COMM_WORLD);
                    active = 0;
                } else {
                    MPI_Recv(&recv_val, 1, MPI_DOUBLE, partner, k, MPI_COMM_WORLD, &status);
                    mypi += recv_val;
                }
            }
            pi = mypi;
        }

        t_end = MPI_Wtime();
        t_total = t_end - t_start;
        t_comp  = t_end - t_start_comp;

        if (myid == 0) {
            double error = fabs(pi - PI25DT);
            char outfile[256];
            FILE *fp;

            if (argc >= 5) strncpy(outfile, argv[4], 255);
            else strncpy(outfile, "par_pi_op.txt", 255);
            outfile[255] = '\0';

            fp = fopen(outfile, "a");
            if (fp) {
                fprintf(fp, "P=%d n=%d p1=%d p2=%d pi=%.16f error=%.16e\n",
                        numprocs, n, p1, p2, pi, error);
                fprintf(fp, "  total_time=%.6f comp_time=%.6f\n", t_total, t_comp);
                fprintf(fp, "SUMMARY: P=%d n=%d tt=%.6f tc=%.6f\n\n",
                        numprocs, n, t_total, t_comp);
                fclose(fp);
            }
        }
    }

    MPI_Finalize();
    return 0;
