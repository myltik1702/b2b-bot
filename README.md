import math
from itertools import permutations
from typing import List, Tuple, Dict

class LogisticsAI:
    def __init__(self, vehicle_capacity: float):
        self.vehicle_capacity = vehicle_capacity

    @staticmethod
    def distance(p1: Tuple[float, float], p2: Tuple[float, float]) -> float:
        return math.hypot(p1[0] - p2[0], p1[1] - p2[1])

    def assign_to_vehicles(self, points: List[Dict]) -> List[List[Dict]]:
        """Жадное распределение заказов по машинам по вместимости."""
        routes = [[]]
        current_load = 0

        for point in points:
            demand = point.get("demand", 1)
            if current_load + demand <= self.vehicle_capacity:
                routes[-1].append(point)
                current_load += demand
            else:
                routes.append([point])
                current_load = demand
        return routes

    @staticmethod
    def tsp_2opt(route: List[Tuple[float, float]], start_point: Tuple[float, float]) -> List[Tuple[float, float]]:
        """Улучшение маршрута 2-opt для TSP. Для небольших наборов работает быстро."""
        # Добавляем старт в начало и конец для удобства
        full_route = [start_point] + route + [start_point]

        def route_length(r):
            return sum(LogisticsAI.distance(r[i], r[i+1]) for i in range(len(r)-1))

        improved = True
        best_route = full_route[:]
        best_len = route_length(best_route)

        while improved:
            improved = False
            for i in range(1, len(best_route)-2):
                for j in range(i+1, len(best_route)-1):
                    new_route = best_route[:i] + best_route[i:j+1][::-1] + best_route[j+1:]
                    new_len = route_length(new_route)
                    if new_len < best_len - 1e-6:
                        best_route = new_route
                        best_len = new_len
                        improved = True
                        break
                if improved:
                    break

        # Возвращаем без дублирующего конца
        return best_route[:-1]

    def optimize_routes(self, depot: Tuple[float, float], points: List[Dict]) -> List[List[Tuple[float, float]]]:
        assigned = self.assign_to_vehicles(points)
        optimized = []
        for route_points in assigned:
            coords = [(p["x"], p["y"]) for p in route_points]
            opt_route = self.tsp_2opt(coords, depot)
            optimized.append(opt_route)
        return optimized


# Пример использования
if __name__ == "__main__":
    depot = (0.0, 0.0)  # Склад
    vehicle_capacity = 10.0

    # Точки доставки: координаты и спрос (вес/объём)
    points = [
        {"name": "A", "x": 1.0, "y": 2.0, "demand": 3},
        {"name": "B", "x": 3.0, "y": 5.0, "demand": 4},
        {"name": "C", "x": -1.0, "y": 4.0, "demand": 2},
        {"name": "D", "x": 2.0, "y": -1.0, "demand": 5},
        {"name": "E", "x": -3.0, "y": -2.0, "demand": 3},
    ]

    ai = LogisticsAI(vehicle_capacity)
    routes = ai.optimize_routes(depot, points)

    total_distance = 0.0
    for i, route in enumerate(routes):
        # Считаем длину с возвратом на склад
        route_with_depot = [depot] + route + [depot]
        dist = sum(LogisticsAI.distance(route_with_depot[j], route_with_depot[j+1])
                   for j in range(len(route_with_depot)-1))
        total_distance += dist
        print(f"Маршрут машины {i+1}: {[p[0] for p in route]}")  # тут выводятся только X для краткости
        print(f"Длина маршрута: {dist:.2f}")

    print(f"\nОбщая длина всех маршрутов: {total_distance:.2f}")
