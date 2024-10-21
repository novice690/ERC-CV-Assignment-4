
import cv2
import mediapipe as mp
import numpy as np
import random

# Initialize MediaPipe Hands
mp_hands = mp.solutions.hands
hands = mp_hands.Hands()
mp_drawing = mp.solutions.drawing_utils

# Game settings
width, height = 1280, 640
player_pos = [320, 440]
enemy_speed = 5
enemy_size = 50
enemy_list = []  # List to store enemy positions
score = 0

# Create random enemy
def create_enemy():
    x = random.randint(0, width - enemy_size)
    return [x, 0]  # Start at the top of the screen

# Move enemies down
def move_enemies(enemy_list):
    for enemy in enemy_list:
        enemy[1] += enemy_speed  # Move enemy down
    # Check if enemy is off-screen
    for enemy in enemy_list[:]:
        if enemy[1] > height:
            enemy_list.remove(enemy)  # Remove off-screen enemy
            return 1  # Increment score for each enemy that goes off-screen
    return 0

# Check for collisions
def check_collision(player_pos, enemy_list):
    player_x, player_y = player_pos
    for enemy in enemy_list:
        enemy_x, enemy_y = enemy
        if (player_x < enemy_x + enemy_size and
            player_x + enemy_size > enemy_x and
            player_y < enemy_y + enemy_size and
            player_y + enemy_size > enemy_y):
            return True  # Collision detected
    return False

# Initialize webcam
cap = cv2.VideoCapture(0)

while True:
    ret, frame = cap.read()
    if not ret:
        break

    frame = cv2.flip(frame, 1)
    rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)

    # Process the frame with MediaPipe
    results = hands.process(rgb_frame)

    # Get coordinates of the index finger tip (landmark 8)
    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            # Draw hand landmarks
            mp_drawing.draw_landmarks(frame, hand_landmarks, mp_hands.HAND_CONNECTIONS)

            # Get the position of the index finger tip
            index_finger_tip = hand_landmarks.landmark[mp_hands.HandLandmark.INDEX_FINGER_TIP]
            player_pos[0] = int(index_finger_tip.x * width) - enemy_size // 2  # Center the player under the finger

    # Move player to stay within screen bounds
    player_pos[0] = max(0, min(player_pos[0], width - enemy_size))

    # Add new enemies
    if random.randint(0, 100) < 5:  # Adjust probability as needed
        enemy_list.append(create_enemy())

    # Move enemies
    score += move_enemies(enemy_list)

    # Check for collision
    if check_collision(player_pos, enemy_list):
        cv2.putText(frame, "Game Over!", (width // 2 - 100, height // 2), cv2.FONT_HERSHEY_SIMPLEX, 2, (0, 0, 255), 3, cv2.LINE_AA)
        break

    # Draw game elements
    cv2.rectangle(frame, (player_pos[0], player_pos[1]), (player_pos[0] + enemy_size, player_pos[1] + enemy_size), (0, 255, 0), -1)
    for enemy in enemy_list:
        cv2.rectangle(frame, (enemy[0], enemy[1]), (enemy[0] + enemy_size, enemy[1] + enemy_size), (255, 0, 0), -1)

    # Display score on the frame
    cv2.putText(frame, f'Score: {score}', (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2, cv2.LINE_AA)

    cv2.imshow("Object Dodging Game", frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()

