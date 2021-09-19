### GSP324: Explore Machine Learning Models with Explainable AI: Challenge Lab :-

----------------------------------------------------------------------------------------------------------------------------------------------

Follow on Twitter @techiezilla , Instagram @techiezilla.



model = Sequential()

model.add(layers.Dense(200, input_shape=(input_size,), activation='relu'))

model.add(layers.Dense(50, activation='relu'))

model.add(layers.Dense(20, activation='relu'))

model.add(layers.Dense(1, activation='sigmoid'))

model.compile(loss='mean_squared_error', optimizer='adam', metrics=['accuracy'])

model.fit(train_data, train_labels, epochs=10, batch_size=2048, validation_split=0.1)



limited_model = Sequential()

limited_model.add(layers.Dense(200, input_shape=(input_size,), activation='relu'))

limited_model.add(layers.Dense(50, activation='relu'))

limited_model.add(layers.Dense(20, activation='relu'))

limited_model.add(layers.Dense(1, activation='sigmoid'))

limited_model.compile(loss='mean_squared_error', optimizer='adam', metrics=['accuracy'])

limited_model.fit(limited_train_data, limited_train_labels, epochs=10, batch_size=2048, validation_split=0.1)



### Fill out the info :-

GCP_PROJECT = '# TODO'
MODEL_BUCKET = 'gs:// #TODO'
MODEL_NAME = 'complete_model' #do not modify
LIM_MODEL_NAME = 'limited_model' #do not modify
VERSION_NAME = 'v1'
REGION = 'us-central1'


!gcloud ai-platform models create $MODEL_NAME --regions $REGION


!gcloud ai-platform versions create $VERSION_NAME \
--model=$MODEL_NAME \
--framework='TENSORFLOW' \
--runtime-version=2.1 \
--origin=$MODEL_BUCKET/saved_model/my_model \
--staging-bucket=$MODEL_BUCKET \
--python-version=3.7


!gcloud ai-platform models create $LIM_MODEL_NAME --regions $REGION


!gcloud ai-platform versions create $VERSION_NAME \
--model=$LIM_MODEL_NAME \
--framework='TENSORFLOW' \
--runtime-version=2.1 \
--origin=$MODEL_BUCKET/saved_limited_model/my_limited_model \
--staging-bucket=$MODEL_BUCKET \
--python-version=3.7



### Congratulations!

Follow on Twitter @techiezilla , Instagram @techiezilla.
