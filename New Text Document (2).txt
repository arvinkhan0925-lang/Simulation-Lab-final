!sudo apt-get install python3-pygame -y
!pip install pygame opencv-python

import pygame
import os
import cv2
import random
from IPython.display import Video

# --- Initialize pygame (headless for Colab) ---
pygame.display.init()
pygame.font.init()

WIDTH, HEIGHT = 640, 480
screen = pygame.Surface((WIDTH, HEIGHT))
clock = pygame.time.Clock()

# --- Initial ball setup ---
balls = [
    {
        "x": WIDTH // 2,
        "y": HEIGHT // 2,
        "dx": 5,
        "dy": 4,
        "color": (255, 80, 80),
        "radius": 25,
        "recent_hit": False   # Prevents multiple splits per frame
    }
]

frames = []
num_frames = 300

for frame in range(num_frames):
    screen.fill((30, 30, 30))
    new_balls = []

    for ball in balls:
        ball["x"] += ball["dx"]
        ball["y"] += ball["dy"]

        hit_wall = False

        # Check for wall collisions
        if ball["x"] - ball["radius"] < 0 or ball["x"] + ball["radius"] > WIDTH:
            ball["dx"] = -ball["dx"]
            hit_wall = True
        if ball["y"] - ball["radius"] < 0 or ball["y"] + ball["radius"] > HEIGHT:
            ball["dy"] = -ball["dy"]
            hit_wall = True

        # Only split once per hit, then wait until no longer touching wall
        if hit_wall and not ball["recent_hit"]:
            # Create a new colored ball
            new_ball = {
                "x": ball["x"],
                "y": ball["y"],
                "dx": -ball["dx"],
                "dy": random.choice([-1, 1]) * abs(ball["dy"]),
                "color": tuple(random.randint(80, 255) for _ in range(3)),
                "radius": ball["radius"],
                "recent_hit": True
            }
            # Change color of the original ball too
            ball["color"] = tuple(random.randint(80, 255) for _ in range(3))
            new_balls.append(new_ball)
            ball["recent_hit"] = True
        elif not hit_wall:
            ball["recent_hit"] = False  # Reset hit status when away from wall

        # Draw the ball
        pygame.draw.circle(screen, ball["color"], (int(ball["x"]), int(ball["y"])), ball["radius"])

    # Add new balls
    balls.extend(new_balls)

    # Limit max number of balls for performance
    if len(balls) > 8:
        balls = balls[-8:]

    # Save frame for video
    frame_data = pygame.surfarray.array3d(screen)
    frame_data = frame_data.transpose([1, 0, 2])
    frames.append(frame_data)

    clock.tick(60)

# --- Save video output ---
out = cv2.VideoWriter("bouncing_split_balls_fixed.mp4", cv2.VideoWriter_fourcc(*'mp4v'), 60, (WIDTH, HEIGHT))
for f in frames:
    out.write(f)
out.release()

print("✅ Clean video ready: bouncing_split_balls_fixed.mp4")

Video("bouncing_split_balls_fixed.mp4")
