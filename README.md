import random
import numpy as np
import pandas as pd

num_jugadores = 12

jugadores = []

for i in range(num_jugadores):
    jugadores.append({
        "jugador": f"Jugador {i+1}",
        "edad": random.randint(16, 21),
        "goles_90": random.uniform(0.0, 0.50),
        "asist_90": random.uniform(0.0, 0.40),
        "pase": random.uniform(70, 90),
        "tackles": random.uniform(0.5, 2.5)
    })

df = pd.DataFrame(jugadores)

def crecimiento_aleatorio(edad):
    mu = (22 - edad) * 0.03
    sd = 0.05

    u1 = random.random()
    u2 = random.random()

    normal = ( (-2 * np.log(u1)) ** 0.5 ) * np.cos(2 * np.pi * u2)
    crecimiento = mu + sd * normal

    return max(-0.25, min(crecimiento, 0.50))

simulaciones = 4000
resultados = []

for idx, row in df.iterrows():

    scores = []

    for _ in range(simulaciones):
        g = crecimiento_aleatorio(row["edad"])

        goles = row["goles_90"] * (1 + g)
        asist = row["asist_90"] * (1 + g)
        pase = row["pase"] * (1 + g)
        tackles = row["tackles"] * (1 + g)

        score = (goles * 60) + (asist * 40) + (pase * 0.7) + (tackles * 10)
        scores.append(score)

    scores = np.array(scores)

    resultados.append({
        "jugador": row["jugador"],
        "score_prom": round(scores.mean(), 2),
        "score_p5": round(np.percentile(scores, 5), 2),
        "score_p95": round(np.percentile(scores, 95), 2),
        "prob_exito(>=75)": round(np.mean(scores >= 75), 3)
    })

tabla = pd.DataFrame(resultados).sort_values("prob_exito(>=75)", ascending=False)

print("\n======= TABLA DE RESULTADOS ORDENADA =======\n")
print(tabla.to_string(index=False))
print("\n============================================")
