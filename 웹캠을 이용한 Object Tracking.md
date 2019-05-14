웹캠을 이용한 Object Tracking

~~~python
import cv2
import numpy as np

cap = cv2.VideoCapture(0)
cap.set(3, 640) #WIDTH
cap.set(4, 480) #HEIGHT


output_size = (375, 667) # (width, height)
fit_to = 'height'


# initialize tracker
OPENCV_OBJECT_TRACKERS = {
  "csrt": cv2.TrackerCSRT_create,
  "kcf": cv2.TrackerKCF_create,
  "boosting": cv2.TrackerBoosting_create,
  "mil": cv2.TrackerMIL_create,
  "tld": cv2.TrackerTLD_create,
  "medianflow": cv2.TrackerMedianFlow_create,
  "mosse": cv2.TrackerMOSSE_create
}

tracker = OPENCV_OBJECT_TRACKERS['csrt']()

# global variables
top_bottom_list, left_right_list = [], []
count = 0

# main
ret, img = cap.read()

cv2.namedWindow('Select Window')
cv2.imshow('Select Window', img)

# select ROI
rect = cv2.selectROI('Select Window', img, fromCenter=False, showCrosshair=True)
cv2.destroyWindow('Select Window')

# initialize tracker
tracker.init(img, rect)

while True:
  count += 1
  # read frame from video
  ret, img = cap.read()

  if not ret:
    exit()

  # update tracker and get position from new frame
  success, box = tracker.update(img)
  # if success:
  left, top, w, h = [int(v) for v in box]
  right = left + w
  bottom = top + h

  # save sizes of image
  top_bottom_list.append(np.array([top, bottom]))
  left_right_list.append(np.array([left, right]))

  # use recent 10 elements for crop (window_size=10)
  if len(top_bottom_list) > 10:
    del top_bottom_list[0]
    del left_right_list[0]

  # compute moving average
  avg_height_range = np.mean(top_bottom_list, axis=0).astype(np.int)
  avg_width_range = np.mean(left_right_list, axis=0).astype(np.int)
  avg_center = np.array([np.mean(avg_width_range), np.mean(avg_height_range)]) # (x, y)
  # compute scaled width and height
  scale = 1.3
  avg_height = (avg_height_range[1] - avg_height_range[0]) * scale
  avg_width = (avg_width_range[1] - avg_width_range[0]) * scale

  # compute new scaled ROI
  avg_height_range = np.array([avg_center[1] - avg_height / 2, avg_center[1] + avg_height / 2])
  avg_width_range = np.array([avg_center[0] - avg_width / 2, avg_center[0] + avg_width / 2])

  # fit to output aspect ratio
  if fit_to == 'width':
    avg_height_range = np.array([
      avg_center[1] - avg_width * output_size[1] / output_size[0] / 2,
      avg_center[1] + avg_width * output_size[1] / output_size[0] / 2
    ]).astype(np.int).clip(0, 9999)

    avg_width_range = avg_width_range.astype(np.int).clip(0, 9999)
  elif fit_to == 'height':
    avg_height_range = avg_height_range.astype(np.int).clip(0, 9999)

    avg_width_range = np.array([
      avg_center[0] - avg_height * output_size[0] / output_size[1] / 2,
      avg_center[0] + avg_height * output_size[0] / output_size[1] / 2
    ]).astype(np.int).clip(0, 9999)

  # visualize
  pt1 = (int(left), int(top))
  pt2 = (int(right), int(bottom))
  cv2.rectangle(img, pt1, pt2, (255, 255, 255), 3)

  cv2.imshow('img', img)

  if cv2.waitKey(1) == ord('q'):
    break

# release everything
cap.release()
out.release()
cv2.destroyAllWindows()
~~~



웹캠을 키고 첫번째 이미지에서 사용자가 사각형을 지정한다.

지정한 위치의 물체를 트래킹하는 코드이다. 



직캠을 만드는 프로그램에서 비디오로 편집되는 부분을 웹캡으로 바꿨다





