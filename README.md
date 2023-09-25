# FitMeIn

## Get matched with a workout buddy
Health if often the most important aspect of ones life but taking care of it can often feel like a daunting task, especially when it comes to exercising.  To most people, workout doesn't come naturally and it is for those people that FitMeIn was made.  All you have to do is to pick what kind of activity you enjoy and you will get matched to a buddy that wants to join you so you can exercise together.  By bringing together people, we hope to give them that extra motivation needed to lead a healthy life (and make friends in the process!).

## Description & Motivation
FitMeIn was the third project in the General Assembly SEI Course.  This project focused on applying the Django Framework and Postgre SQL.  Additionally we opted to implement a Geolocation API. This app was a group project that taught us how to pair and mob program and practice the pull/push workflow in team work.

![Python](https://img.shields.io/badge/Python-3776AB.svg?style=for-the-badge&logo=Python&logoColor=white)
![Django](https://img.shields.io/badge/Django-092E20.svg?style=for-the-badge&logo=Django&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E.svg?style=for-the-badge&logo=JavaScript&logoColor=black)
![Bootstrap](https://img.shields.io/badge/Bootstrap-7952B3.svg?style=for-the-badge&logo=Bootstrap&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-E34F26.svg?style=for-the-badge&logo=HTML5&logoColor=white)
![Heroku](https://img.shields.io/badge/Heroku-430098.svg?style=for-the-badge&logo=Heroku&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1.svg?style=for-the-badge&logo=PostgreSQL&logoColor=white)
![AWS](https://img.shields.io/badge/Amazon%20AWS-232F3E.svg?style=for-the-badge&logo=Amazon-AWS&logoColor=white)

### Check it out! [FitMeIn](https://fit-me-in-7fcf0f4ba962.herokuapp.com/)

## Team Work
Our Team: [Angelica Sandrini](https://github.com/gellisun) | [Hannah Curran](https://github.com/hannahcurran) | [James Carter](https://github.com/JamesC215) | [Lucas Neno](https://github.com/casneno)

From the get-go the entire team got together very well.  As scrum master I divised a series of brainstorming sessions for us to decide on the group name and project, followed by a wireframe building session where the synergy showed very well.

## Brief
- Connect to and perform data operations on a PostgreSQL database (the default SQLLite3 database is not acceptable).
- If consuming an API, have at least one data entity (Model) in addition to the built-in User model.
- If not consuming an API, have at least two data entities (Models) in addition to the built-in User model.
- Have full-CRUD data operations across any combination of the app's models (excluding the User model).
- Authenticate users using Django's built-in authentication.
- Implement authorization by restricting access to the Creation, Updating & Deletion of data resources.

## Planning

### Brainstorming
![Initial brainstorming on the project](/main_app/static/images/README/Brainstorming.png "Initial brainstorming on the project")

### Wireframe
![Mobile](/main_app/static/images/README/mobile-wireframe.png "Wireframe for mobile")<br>

During the planning we also decided that we would be working with a mix of pair/mob programming and solo programming depending on the needs and functionalities.

![Web](/main_app/static/images/README/web-wireframe.png "Wireframe for web")
### ERD
![ERD](/main_app/static/images/README/erd.png "ERD")

## Building the Code

Our first approach to coding started in the planning phase where we defined what feature each person would implement.  I took charge of the API for most the project.  In the beginning we needed to define the models and establish the one-to-one realtionships. After doing so, each one took charge of coding a different section of the app. O began working on the API.


```Python
@login_required
def profile(request):
  try:
    profile = Profile.objects.get(user=request.user)
    context = {'profile': profile}
    if profile:
        profile_form = ProfileForm(instance=profile)
        comments = Comment.objects.filter(user=request.user)
        context['profile_form']=profile_form
        context['comments']=comments
    return render(request, 'user/profile.html', context)
  except Profile.DoesNotExist:
    return redirect('create_profile')


# ---------------- Sign-Up ------------------------


def signup(request):
  error_message = ''
  if request.method == 'POST':
    form = UserCreationForm(request.POST)
    if form.is_valid():
      user = form.save()
      login(request, user)
      return redirect('create_profile')
    else:
      error_message = 'Invalid sign up - try again!'
  form = UserCreationForm()
  context = {'form': form, 'error_message': error_message}
  return render(request, 'registration/signup.html', context)


#-------------Create Profile----------------


class ProfileCreate(CreateView):
  model = Profile
  template_name = 'user/create_profile.html'
  fields = ['age', 'gender', 'location', 'is_couch_potato', 'favorites', 'latitude', 'longitude', 'is_active']
  success_url = reverse_lazy('profile')  # Replace 'profile-detail' with your actual URL pattern

  def form_valid(self, form):
      print('form_valid being executed')
      form.instance.user = self.request.user
      print(form)
      return super().form_valid(form)
```
### API
Amongst one of the features is the API call: our program matches two or more users using their GPS/IP location by means of the HTLM5 Geolocation API.  When first prompted, the user provides data on which activities he would like to perform and this information is stored in his profile.  The program then makes a JavaScript call to the API that takes the user's 'latitude' and 'longitude', it then passes on those values through an URL to the back-end which are then parsed and stored in the database.  Finally, a filter is applied to every active profile in the database that has selected similar activities to that of the user, and calculates the distance between these profiles and the user, returning a table of profiles within a certain range that have the same activity interests as our user.

```JavaScript
const displayCoord = document.getElementById("displayCoord")
const latitudeDisplay = document.getElementById("latitude")
const longitudeDisplay = document.getElementById("longitude")

displayCoord.addEventListener("click", getLocation)

function getLocation(event) {
    event.preventDefault()
    console.log('click')
    if ("geolocation" in navigator) {
        navigator.geolocation.getCurrentPosition(success, error);
    } else {
        alert("Geolocation is not available in your browser.");
    }
}

function success(position) {
    const latitude = position.coords.latitude;
    const longitude = position.coords.longitude;
    console.log('success')
    console.log(latitude)

    window.location.assign(`http://localhost:8000/my_match/${latitude}/${longitude}/`)

}
```
```Python
def haversine(lat1, lon1, lat2, lon2):
  R = 6371
  dist_lat = math.radians(lat2 - lat1)
  dist_lon = math.radians(lon2 - lon1)
  a = math.sin(dist_lat / 2) * math.sin(dist_lat / 2) + math.cos(math.radians(lat1)) * math.cos(math.radians(lat2)) * math.sin(dist_lon / 2) * math.sin(dist_lon / 2)
  c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
  distance = R*c
  return distance

def find_match(request, profile_id):
  profile = Profile.objects.get(id=profile_id)
  user_latitude = profile.latitude
  user_longitude = profile.longitude
  user_chosen_activities = profile.chosen_activities
  active_profiles = Profile.objects.filter(is_active=True, chosen_activities__contains=user_chosen_activities).exclude(id=profile.id)
  matched_profiles = []
  matched_distance = [] 
  #Check if haversine distance is within a range (5.0km)
  for profile in active_profiles:
    distance = haversine(user_latitude, user_longitude, profile.latitude, profile.longitude)  
    if distance < 5000.0:
      matched_profiles.append(profile)
      matched_distance.append(distance)
  print(matched_profiles)
  print(matched_distance)
  return render(request, 'user/my_matches.html', {'profile':profile, 'matched_profiles':matched_profiles, 'matched_distance':matched_distance, 'user_latitude':user_latitude, 'user_longitude':user_longitude})
```

## Challenges
Most of the challenges in this project revolved around the pull/push and migrate aspect of team work and PostgreSQL.  More often than not we got conflicts in the beginning and dedicated a lot of time to debugging and resetting the database.  Also, our final relationship between entities wasn't correct so our model was flawed and whe testing, we weren't able to relate the user'd unique Id to the comment section.

## Wins
Managing to coordinate the workflow without merging errors and migration issues was definetely a win for us.  Our group communication was also very good from the start and that was essential for us to understand what was it that we were doing wrong whenever a wrkflow issue would arrise. As a team I felt that we were very supportive of one another and I really enjoyed debugging in pair programming.
Personally, the logic behind calculating the Geolocation and implementing the HTML5 Geolocation API were two of my greatest wins, since it was the first time I implemented an API.

## Key Learnings
I would say that the biggest takeaway would be Git and GitHub workflow as a team. I believe we grew a lot during that week and learnt a lot from each other, especially when it came to Bootstrap, which was new for everyone and debugging Django messages.
Learning and making use of the Python and Django framework was great!
Implementing an API was my personal key learning from this project, especially when it comes to reading thoroughly the related documentation.

## Future Improvements
- We would like the app show all the connections made and improve its social feature.
- We would like it to also be a place where, based on the user's location, fitness events in the area can be suggested.
- We would like to add the possiblity to also find recipes and eating habits and suggestions to improve the user's health.
- We finally but not lastly would like to improve its style and UI/UX.
