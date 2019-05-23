# dlib-opencv_pose_estimation
Extract 14 key points from dlib 68 face key points and estimate face pose by cv2.solvePnP

## Steps:
- 利用dlib人脸检测 dlib.get_frontal_face_detector 以及`dlib.shape_predictor(r"D:\Programming workspaces\LocalGithub\mtcnn-caffe-master\demo\shape_predictor_68_face_landmarks.dat")` 获取人脸68个关键点  
其中`shape_predictor_68_face_landmarks.dat`是可从网上下载的官方模型  

**在cmcc的AI平台上可以直接得到dlib人脸关键点的可视化结果：[地址](http://jiutian.cmri.cn/demo/demo_face_location)**  

- 从检测到的人脸中获取最大的人脸 `def _largest_face(dets)`
- 从dlib 68个关键点中抽取14个  
``````
image_points = np.array([
                        (landmark_shape.part(17).x, landmark_shape.part(17).y),  #17 left brow left corner
                        (landmark_shape.part(21).x, landmark_shape.part(21).y),  #21 left brow right corner
                        (landmark_shape.part(22).x, landmark_shape.part(22).y),  #22 right brow left corner
                        (landmark_shape.part(26).x, landmark_shape.part(26).y),  #26 right brow right corner
                        (landmark_shape.part(36).x, landmark_shape.part(36).y),  #36 left eye left corner
                        (landmark_shape.part(39).x, landmark_shape.part(39).y),  #39 left eye right corner
                        (landmark_shape.part(42).x, landmark_shape.part(42).y),  #42 right eye left corner
                        (landmark_shape.part(45).x, landmark_shape.part(45).y),  #45 right eye right corner
                        (landmark_shape.part(31).x, landmark_shape.part(31).y),  #31 nose left corner
                        (landmark_shape.part(35).x, landmark_shape.part(35).y),  #35 nose right corner
                        (landmark_shape.part(48).x, landmark_shape.part(48).y),  #48 mouth left corner
                        (landmark_shape.part(54).x, landmark_shape.part(54).y),  #54 mouth right corner
                        (landmark_shape.part(57).x, landmark_shape.part(57).y),  #57 mouth central bottom corner
                        (landmark_shape.part(8).x, landmark_shape.part(8).y),  #8 chin corner
                    ], dtype="double")
``````  
- 利用opencv solvePnP 估计脸部姿态, 得到rotation vector和 transition vector  

3D标准脸部14个关键点模板： 
``````
model_points = np.array([
                        (6.825897, 6.760612, 4.402142),     #33 left brow left corner
                        (1.330353, 7.122144, 6.903745),     #29 left brow right corner
                        (-1.330353, 7.122144, 6.903745),    #34 right brow left corner
                        (-6.825897, 6.760612, 4.402142),    #38 right brow right corner
                        (5.311432, 5.485328, 3.987654),     #13 left eye left corner
                        (1.789930, 5.393625, 4.413414),     #17 left eye right corner
                        (-1.789930, 5.393625, 4.413414),    #25 right eye left corner
                        (-5.311432, 5.485328, 3.987654),    #21 right eye right corner
                        (2.005628, 1.409845, 6.165652),     #55 nose left corner
                        (-2.005628, 1.409845, 6.165652),    #49 nose right corner
                        (2.774015, -2.080775, 5.048531),    #43 mouth left corner
                        (-2.774015, -2.080775, 5.048531),   #39 mouth right corner
                        (0.000000, -3.116408, 6.097667),    #45 mouth central bottom corner
                        (0.000000, -7.415691, 4.070434)     #6 chin corner
                    ])
``````  
相机相关参数：
``````
    focal_length = img_size[1]
    center = (img_size[1]/2, img_size[0]/2)
    camera_matrix = np.array(
                             [[focal_length, 0, center[0]],
                             [0, focal_length, center[1]],
                             [0, 0, 1]], dtype = "double"
                             )

    dist_coeffs = np.array([7.0834633684407095e-002, 6.9140193737175351e-002, 0.0, 0.0, -1.3073460323689292e+000],dtype= "double")
``````
- 利用公式将rotation vector转化为欧拉角yaw， pitch 和 roll
