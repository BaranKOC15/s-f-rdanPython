import cv2
import mediapipe as mp

mp_drawing = mp.solutions.drawing_utils
mp_pose = mp.solutions.pose

# Videodan kareler al
cap = cv2.VideoCapture(0)

jump_counter = 0
above_line = False
below_line = True
some_threshold=0.1

# Pencere olu_tur ve boyutunu ayarla
cv2.namedWindow('MediaPipe Pose', cv2.WINDOW_NORMAL)
cv2.resizeWindow('MediaPipe Pose', 800, 600)  # Geni_lik ve y�kseklii ayarla

# 0nsan pozisyonunu alg1la
with mp_pose.Pose(min_detection_confidence=0.5, min_tracking_confidence=0.5) as pose:
    while cap.isOpened():
        ret, frame = cap.read()
        
        # RGB'ye �evir
        image = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        image.flags.writeable = False
      
        # Alg1lama
        results = pose.process(image)
    
        # �izim i�in g�r�nt�y� geri d�nd�r
        image.flags.writeable = True
        image = cv2.cvtColor(image, cv2.COLOR_RGB2BGR)
      
        # �izim
        mp_drawing.draw_landmarks(image, results.pose_landmarks, mp_pose.POSE_CONNECTIONS)
        
        # Z1plama alg1lama
        try:
            landmarks = results.pose_landmarks.landmark
            
            # Ba_ y�ksekliini kontrol et
            head = landmarks[mp_pose.PoseLandmark.NOSE]

            # Z1plama alg1lama
            if head.y > some_threshold:
                if not above_line and below_line:
                    above_line = True
                    below_line = False
            elif head.y <= some_threshold:
                if not below_line:
                    below_line = True
                    if above_line:
                        jump_counter += 1
                        above_line = False

            # Ba_ y�ksekliini yazd1r
            #cv2.putText(image, f"Head Height: {head.y}", (10, 70), cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2, cv2.LINE_AA)

        except:
            pass

        # �izgi �iz
        cv2.line(image, (0, int(some_threshold * image.shape[0])), (image.shape[1], int(some_threshold * image.shape[0])), (0, 0, 255), 2)
        
        # Z1plama say1s1n1 yazd1r
        cv2.putText(image, f"Ziplama: {jump_counter}", (0, 30), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2, cv2.LINE_AA)

        cv2.imshow('MediaPipe Pose', image)
        if cv2.waitKey(10) & 0xFF == ord('q'):
            break

cap.release()
cv2.destroyAllWindows()r
