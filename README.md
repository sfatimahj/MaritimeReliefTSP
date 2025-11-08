import java.util.*;

public class MaritimeReliefRouteOptimization {

    static int[][] distanceMatrix = {
        {0, 10, 20, 25},   // Port A distances
        {10, 0, 25, 15},   // Port B distances
        {20, 25, 0, 20},   // Relief Center C distances
        {25, 15, 20, 0}    // Relief Center D distances
    };

    static String[] locations = {"Port A", "Port B", "Relief Center C", "Relief Center D"};

    // --- Greedy TSP ---
    public static void greedyTSP(int[][] dist) {
        int n = dist.length;
        boolean[] visited = new boolean[n];
        int current = 0, total = 0;
        visited[current] = true;
        StringBuilder path = new StringBuilder(locations[current]);

        for (int step = 1; step < n; step++) {
            int next = -1, best = Integer.MAX_VALUE;
            for (int j = 0; j < n; j++) {
                if (!visited[j] && dist[current][j] < best) {
                    best = dist[current][j];
                    next = j;
                }
            }
            visited[next] = true;
            total += dist[current][next];
            current = next;
            path.append(" -> ").append(locations[current]);
        }
        total += dist[current][0];
        path.append(" -> ").append(locations[0]);

        System.out.println("Greedy TSP Route: " + path + " | Total Distance: " + total + " nm");
    }

    // --- Dynamic Programming TSP ---
    public static void dynamicProgrammingTSP(int[][] dist) {
        int n = dist.length;
        int[][] dp = new int[1 << n][n];
        int[][] parent = new int[1 << n][n];
        for (int[] row : dp) Arrays.fill(row, Integer.MAX_VALUE / 2);
        for (int[] row : parent) Arrays.fill(row, -1);

        dp[1][0] = 0;
        for (int mask = 1; mask < (1 << n); mask++) {
            for (int u = 0; u < n; u++) {
                if ((mask & (1 << u)) == 0) continue;
                for (int v = 0; v < n; v++) {
                    if ((mask & (1 << v)) != 0) continue;
                    int nextMask = mask | (1 << v);
                    int newCost = dp[mask][u] + dist[u][v];
                    if (newCost < dp[nextMask][v]) {
                        dp[nextMask][v] = newCost;
                        parent[nextMask][v] = u;
                    }
                }
            }
        }

        int best = Integer.MAX_VALUE, last = -1, fullMask = (1 << n) - 1;
        for (int u = 0; u < n; u++) {
            int cost = dp[fullMask][u] + dist[u][0];
            if (cost < best) {
                best = cost;
                last = u;
            }
        }

        LinkedList<String> rev = new LinkedList<>();
        int mask = fullMask, cur = last;
        while (cur != -1) {
            rev.addFirst(locations[cur]);
            int p = parent[mask][cur];
            mask ^= (1 << cur);
            cur = p;
        }
        rev.addFirst(locations[0]);
        rev.addLast(locations[0]);

        System.out.println("Dynamic Programming TSP Route: " + String.join(" -> ", rev)
                + " | Total Distance: " + best + " nm");
    }

    // --- Backtracking TSP ---
    public static void backtrackingTSP(int[][] dist) {
        int n = dist.length;
        boolean[] visited = new boolean[n];
        visited[0] = true;
        StringBuilder bestPath = new StringBuilder();
        int[] bestCost = {Integer.MAX_VALUE};
        tspBacktracking(0, dist, visited, n, 1, 0, new StringBuilder(locations[0]), bestPath, bestCost);

        System.out.println("Backtracking TSP Route: " + bestPath + " | Total Distance: " + bestCost[0] + " nm");
    }

    private static void tspBacktracking(int pos, int[][] dist, boolean[] visited, int n, int count, int cost,
                                        StringBuilder path, StringBuilder bestPath, int[] bestCost) {
        if (count == n) {
            int total = cost + dist[pos][0];
            StringBuilder full = new StringBuilder(path).append(" -> ").append(locations[0]);
            if (total < bestCost[0]) {
                bestCost[0] = total;
                bestPath.setLength(0);
                bestPath.append(full);
            }
            return;
        }
        for (int city = 0; city < n; city++) {
            if (!visited[city]) {
                int newCost = cost + dist[pos][city];
                if (newCost >= bestCost[0]) continue;
                visited[city] = true;
                int lenBefore = path.length();
                path.append(" -> ").append(locations[city]);
                tspBacktracking(city, dist, visited, n, count + 1, newCost, path, bestPath, bestCost);
                path.setLength(lenBefore);
                visited[city] = false;
            }
        }
    }

    // --- Divide & Conquer TSP ---
    public static void divideAndConquerTSP(int[][] dist) {
        int n = dist.length;
        boolean[] visited = new boolean[n];
        visited[0] = true;
        StringBuilder bestPath = new StringBuilder();
        int[] bestCost = {Integer.MAX_VALUE};
        divideAndConquerHelper(0, visited, 0, dist, n, new StringBuilder(locations[0]), bestPath, bestCost);
        System.out.println("Divide & Conquer TSP Route: " + bestPath + " | Total Distance: " + bestCost[0] + " nm");
    }

    private static void divideAndConquerHelper(int pos, boolean[] visited, int currentCost,
                                               int[][] dist, int n, StringBuilder path,
                                               StringBuilder bestPath, int[] bestCost) {
        boolean allVisited = true;
        for (boolean v : visited) if (!v) allVisited = false;
        if (allVisited) {
            int total = currentCost + dist[pos][0];
            StringBuilder full = new StringBuilder(path).append(" -> ").append(locations[0]);
            if (total < bestCost[0]) {
                bestCost[0] = total;
                bestPath.setLength(0);
                bestPath.append(full);
            }
            return;
        }
        for (int city = 0; city < n; city++) {
            if (!visited[city]) {
                int newCost = currentCost + dist[pos][city];
                if (newCost >= bestCost[0]) continue;
                visited[city] = true;
                int lenBefore = path.length();
                path.append(" -> ").append(locations[city]);
                divideAndConquerHelper(city, visited, newCost, dist, n, path, bestPath, bestCost);
                path.setLength(lenBefore);
                visited[city] = false;
            }
        }
    }

    // --- Insertion Sort ---
    public static void insertionSort(int[] arr) {
        for (int i = 1; i < arr.length; i++) {
            int key = arr[i];
            int j = i - 1;
            while (j >= 0 && arr[j] > key) {
                arr[j + 1] = arr[j];
                j--;
            }
            arr[j + 1] = key;
        }
    }

    // --- Binary Search ---
    public static int binarySearch(int[] arr, int target) {
        int lo = 0, hi = arr.length - 1;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            if (arr[mid] == target) return mid;
            else if (arr[mid] < target) lo = mid + 1;
            else hi = mid - 1;
        }
        return -1;
    }

    // --- Main Driver ---
    public static void main(String[] args) {
        // TSP Routes
        greedyTSP(distanceMatrix);
        dynamicProgrammingTSP(distanceMatrix);
        backtrackingTSP(distanceMatrix);
        divideAndConquerTSP(distanceMatrix);

        // Sorting & Searching Example
        int[] arr = {8, 3, 5, 1, 9, 2};
        insertionSort(arr);
        System.out.println("Sorted Array: " + Arrays.toString(arr));
        System.out.println("Binary Search for 5: Index " + binarySearch(arr, 5));
    }
}
