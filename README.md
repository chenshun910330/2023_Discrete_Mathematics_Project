---------------------------------優化前虛擬碼----------------------------------------------------
Algorithm is_transitive_naive(Matrix M, n):
    For i from 0 to n-1:
        For k from 0 to n-1:
            If M[i][k] is True:
                For j from 0 to n-1:
                    If M[k][j] is True AND M[i][j] is False:
                        Return False
    Return True
---------------------------------優化後虛擬碼----------------------------------------------------
Algorithm is_transitive_opt(Rows R, n):
    For i from 0 to n-1:
        For k from 0 to n-1:
            If the k-th bit of R[i] is 1:
                // Check if R[k] implies R[i]
                If (R[k] bitwise-AND R[i]) != R[k]:
                    Return False
    Return True
-----------------------------------C code--------------------------------------------------
#include <stdio.h>
#include <stdint.h>
#include <stdbool.h>
#include <time.h>
#include <math.h>

// [優化前] 暴力法：使用三層迴圈檢查每一個 (i, k, j) 組合
bool is_transitive_naive(int n, int M[6][6]) {
    for (int i = 0; i < n; i++) {           // 遍歷起點 i
        for (int k = 0; k < n; k++) {       // 遍歷中間點 k
            if (M[i][k] == 1) {             // 如果存在 i -> k 的關係
                for (int j = 0; j < n; j++) { // 遍歷終點 j
                    // 檢查：若 k -> j 存在，但 i -> j 不存在，則違反遞移律
                    if (M[k][j] == 1 && M[i][j] == 0) return false;
                }
            }
        }
    }
    return true; // 通過所有檢查，代表符合遞移律
}

// [優化後] 位元向量法：利用位元運算一次處理多個列元素的比較
bool is_transitive_opt(int n, uint64_t R[]) {
    for (int i = 0; i < n; i++) {           // 遍歷每一行 i
        for (int k = 0; k < n; k++) {       // 檢查第 i 行中是否有連向 k 的邊
            if ((R[i] >> k) & 1ULL) {       // 判斷第 i 個位元向量的第 k 位是否為 1
                // 核心邏輯：若 i -> k，則 k 所能到達的所有點，i 也必須能到達
                // 位元運算：(R[k] & R[i]) == R[k] 代表 R[k] 是 R[i] 的子集
                if ((R[k] & R[i]) != R[k]) return false;
            }
        }
    }
    return true;
}

void run_benchmark(int n) {
    uint64_t total = 1ULL << (n * n);       // 計算 n*n 矩陣所有可能的組合數 (2^(n^2))
    uint64_t count_naive = 0, count_opt = 0;
    int matrix[6][6];
    uint64_t rows[6];

    printf("N = %d (總組合數: %llu)\n", n, total);

    // --- 測試暴力法 ---
    clock_t start_n = clock();              // 記錄起始時間
    for (uint64_t i = 0; i < total; i++) {  // 窮舉所有可能的二元關係
        for (int r = 0; r < n; r++) {       // 將整數 i 轉換為二維陣列矩陣
            for (int c = 0; c < n; c++) {
                matrix[r][c] = (i >> (r * n + c)) & 1;
            }
        }
        if (is_transitive_naive(n, matrix)) count_naive++;
    }
    clock_t end_n = clock();                // 記錄結束時間
    double time_naive = (double)(end_n - start_n) / CLOCKS_PER_SEC;

    // --- 測試優化法 ---
    clock_t start_o = clock();
    for (uint64_t i = 0; i < total; i++) {
        for (int r = 0; r < n; r++) {       // 將整數 i 轉換為位元向量陣列
            rows[r] = (i >> (r * n)) & ((1ULL << n) - 1);
        }
        if (is_transitive_opt(n, rows)) count_opt++;
    }
    clock_t end_o = clock();
    double time_opt = (double)(end_o - start_o) / CLOCKS_PER_SEC;

    // 輸出結果與加速倍率
    printf("  [Naive] 數量: %llu, 時間: %.7f s\n", count_naive, time_naive);
    printf("  [Optim] 數量: %llu, 時間: %.7f s\n", count_opt, time_opt);
    printf("  >> 加速倍率: %.2fx\n\n", time_naive / time_opt);
}

int main() {
    printf("遞移關係計算實驗報告 (n=1 to 5)\n");
    printf("----------------------------------------\n");
    for (int i = 1; i <= 6; i++) {
        run_benchmark(i);                   // 分別對 n=1 到 5 進行測試
    }
    return 0;
}
