# mountain-rescue-drone-ai
import random
import heapq
import matplotlib.pyplot as plt

SIZE = 10
TRIALS = 30

# ---------------------------
# 지도 생성
# ---------------------------
def generate_map():
    return [[random.randint(0, 3) for _ in range(SIZE)] for _ in range(SIZE)]

# ---------------------------
# 조난자 생성
# ---------------------------
def place_target(grid):
    x = random.randint(0, SIZE - 1)
    y = random.randint(0, SIZE - 1)
    grid[y][x] = 4
    return (x, y)

# ---------------------------
# Random 탐색
# ---------------------------
def random_search(start, target):
    x, y = start
    steps = 0

    for _ in range(300):
        dx, dy = random.choice([(1,0), (-1,0), (0,1), (0,-1)])
        nx, ny = x + dx, y + dy

        if 0 <= nx < SIZE and 0 <= ny < SIZE:
            x, y = nx, ny
            steps += 1

            if (x, y) == target:
                return steps

    return 300

# ---------------------------
# Zigzag 탐색
# ---------------------------
def zigzag_search(start, target):
    steps = 0
    direction = 1

    for y in range(SIZE):

        if direction == 1:
            cols = range(SIZE)
        else:
            cols = range(SIZE - 1, -1, -1)

        for x in cols:
            steps += 1

            if (x, y) == target:
                return steps

        direction *= -1

    return steps

# ---------------------------
# A* 탐색
# ---------------------------
def heuristic(a, b):
    return abs(a[0] - b[0]) + abs(a[1] - b[1])

def astar(start, target):

    open_set = []
    heapq.heappush(open_set, (0, start))

    g_score = {start: 0}

    while open_set:

        _, current = heapq.heappop(open_set)

        if current == target:
            return g_score[current]

        x, y = current

        neighbors = [
            (x + 1, y),
            (x - 1, y),
            (x, y + 1),
            (x, y - 1)
        ]

        for neighbor in neighbors:

            nx, ny = neighbor

            if not (0 <= nx < SIZE and 0 <= ny < SIZE):
                continue

            new_cost = g_score[current] + 1

            if neighbor not in g_score or new_cost < g_score[neighbor]:

                g_score[neighbor] = new_cost

                priority = new_cost + heuristic(neighbor, target)

                heapq.heappush(open_set, (priority, neighbor))

    return 300

# ---------------------------
# 실험 실행
# ---------------------------
random_results = []
zigzag_results = []
astar_results = []

for _ in range(TRIALS):

    grid = generate_map()

    target = place_target(grid)

    start = (0, 0)

    random_results.append(
        random_search(start, target)
    )

    zigzag_results.append(
        zigzag_search(start, target)
    )

    astar_results.append(
        astar(start, target)
    )

# ---------------------------
# 평균 계산
# ---------------------------
random_avg = sum(random_results) / TRIALS
zigzag_avg = sum(zigzag_results) / TRIALS
astar_avg = sum(astar_results) / TRIALS

# ---------------------------
# 결과 출력
# ---------------------------
print("\n===== 드론 탐색 알고리즘 비교 =====")
print(f"Random 평균 이동 횟수 : {random_avg:.2f}")
print(f"Zigzag 평균 이동 횟수 : {zigzag_avg:.2f}")
print(f"A* 평균 이동 횟수     : {astar_avg:.2f}")

# ---------------------------
# 그래프 저장
# ---------------------------
plt.figure(figsize=(6,4))

algorithms = ["Random", "Zigzag", "A*"]
scores = [random_avg, zigzag_avg, astar_avg]

plt.bar(algorithms, scores)

plt.title("Drone Search Algorithm Comparison")
plt.xlabel("Algorithm")
plt.ylabel("Average Steps")

plt.tight_layout()

plt.savefig("result.png")

print("\n그래프 저장 완료!")
print("왼쪽 파일 목록에서 result.png를 클릭해서 확인하세요.")
