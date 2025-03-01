import cv2
import mediapipe as mp
import numpy as np
import screen_brightness_control as sbc
from collections import deque


mp_hands = mp.solutions.hands
mp_draw = mp.solutions.drawing_utils
hands = mp_hands.Hands(min_detection_confidence=0.7, min_tracking_confidence=0.7)

cap = cv2.VideoCapture(0)
canvas = np.zeros((480, 640, 3), dtype=np.uint8)  


left_hand_positions = deque(maxlen=10)
shake_threshold = 50


previous_point = None


written_text = ""


while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        print("Failed to capture video. Exiting...")
        break

   
    frame = cv2.flip(frame, 1)
    rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)


    result = hands.process(rgb_frame)
    right_hand_present = False
    left_hand_present = False

    if result.multi_hand_landmarks:
        for idx, hand_landmarks in enumerate(result.multi_hand_landmarks):
           
            handedness = result.multi_handedness[idx].classification[0].label
            if handedness == "Right":
                right_hand_present = True
            elif handedness == "Left":
                left_hand_present = True

       
            mp_draw.draw_landmarks(frame, hand_landmarks, mp_hands.HAND_CONNECTIONS)

          
            h, w, _ = frame.shape
            thumb_tip = hand_landmarks.landmark[mp_hands.HandLandmark.THUMB_TIP]
            index_tip = hand_landmarks.landmark[mp_hands.HandLandmark.INDEX_FINGER_TIP]

            thumb_coords = (int(thumb_tip.x * w), int(thumb_tip.y * h))
            index_coords = (int(index_tip.x * w), int(index_tip.y * h))

            if handedness == "Right":
              
                if previous_point is None:
                    previous_point = index_coords
                else:
                    cv2.line(canvas, previous_point, index_coords, (255, 255, 255), 5)
                    previous_point = index_coords

         
                cv2.putText(frame, "Drawing...", (10, 100),
                            cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)

            if handedness == "Left":
               
                distance = np.linalg.norm(np.array(thumb_coords) - np.array(index_coords))
                brightness = np.interp(distance, [50, 200], [0, 100])
                try:
                    sbc.set_brightness(int(brightness)) 
                except Exception as e:
                    print(f"Could not set brightness: {e}")

         
                cv2.putText(frame, f'Brightness: {int(brightness)}%', (10, 50),
                            cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2)

                palm_coords = hand_landmarks.landmark[mp_hands.HandLandmark.WRIST]
                palm_point = (int(palm_coords.x * w), int(palm_coords.y * h))
                left_hand_positions.append(palm_point)

                if len(left_hand_positions) == left_hand_positions.maxlen:
                    movement = sum(
                        np.linalg.norm(np.array(left_hand_positions[i]) - np.array(left_hand_positions[i - 1]))
                        for i in range(1, len(left_hand_positions))
                    )
                    if movement > shake_threshold:
                        canvas = np.zeros((480, 640, 3), dtype=np.uint8) 
                        previous_point = None  

    else:
        previous_point = None 

   
    combined = cv2.addWeighted(frame, 0.5, canvas, 0.5, 0)

   
    cv2.imshow("Hand Gesture Control", combined)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break


cap.release()
cv2.destroyAllWindows()
