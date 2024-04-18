
## Uplaod on Youtube

```py
pip install google-auth google-auth-oauthlib google-api-python-client

```

```py
from google_auth_oauthlib.flow import InstalledAppFlow
from google.auth.transport.requests import Request
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload

SCOPES = ['https://www.googleapis.com/auth/youtube.upload']
CLIENT_SECRET_FILE = 'client_secret.json'
VIDEO_FILE = 'video.mp4'
THUMBNAIL_FILE = 'thumbnail.jpg'
API_SERVICE_NAME = 'youtube'
API_VERSION = 'v3'

def authenticate():
    flow = InstalledAppFlow.from_client_secrets_file(CLIENT_SECRET_FILE, SCOPES)
    credentials = flow.run_local_server()
    return credentials

def upload_video():
    credentials = authenticate()
    youtube = build(API_SERVICE_NAME, API_VERSION, credentials=credentials)

    request = youtube.videos().insert(
        part='snippet,status',
        body={
            'snippet': {
                'categoryId': '22',  # See https://developers.google.com/youtube/v3/docs/videoCategories/list for category IDs
                'description': 'Video description',
                'title': 'Video title',
                'tags': ['tag1', 'tag2'],
                'thumbnails': {
                    'default': {
                        'url': 'URL to your thumbnail image',
                        'width': 120,
                        'height': 90
                    }
                }
            },
            'status': {
                'privacyStatus': 'private'  # Set privacy status ('public', 'unlisted', or 'private')
            }
        },
        media_body=MediaFileUpload(VIDEO_FILE)
    )

    response = request.execute()
    print(response)

if __name__ == '__main__':
    upload_video()

```



## Video Edit

```py
from flask import Flask, render_template, request, redirect, url_for
from flask_uploads import UploadSet, configure_uploads, VIDEO
from moviepy.editor import VideoFileClip, concatenate_videoclips, TextClip, CompositeVideoClip

app = Flask(__name__)

# Configure file upload
videos = UploadSet('videos', VIDEO)
app.config['UPLOADED_VIDEOS_DEST'] = 'uploads/videos'
configure_uploads(app, videos)

# Route for uploading and editing video
@app.route('/', methods=['GET', 'POST'])
def upload_video():
    if request.method == 'POST' and 'video' in request.files:
        video = request.files['video']
        filename = videos.save(video)
        
        # Edit the video
        original_clip = VideoFileClip(f'uploads/videos/{filename}')
        intro_clip = VideoFileClip('intro.mp4').crossfadein(1)
        outro_clip = VideoFileClip('outro.mp4').crossfadein(1)
        logo = 'logo.png'
        logo_clip = (ImageClip(logo)
                     .set_duration(original_clip.duration)
                     .resize(height=50)  # Resize if needed
                     .margin(right=8, top=8, opacity=0)  # Position logo
                     .set_pos(("right","top")))
        video_with_intro_outro = concatenate_videoclips([intro_clip, original_clip, outro_clip])
        video_with_logo = CompositeVideoClip([video_with_intro_outro, logo_clip])
        
        # Add background music
        bg_music = AudioFileClip('bg_music.mp3').subclip(0, video_with_logo.duration).volumex(0.5)
        final_video = video_with_logo.set_audio(bg_music)
        
        # Save the edited video
        final_filename = f'edited_{filename}'
        final_video.write_videofile(f'uploads/videos/{final_filename}')
        
        return redirect(url_for('uploaded_file', filename=final_filename))
    return render_template('upload.html')

# Route to display the edited video
@app.route('/uploads/<filename>')
def uploaded_file(filename):
    return render_template('uploaded.html', filename=filename)

if __name__ == '__main__':
    app.run(debug=True)

```
