---
layout: post
title: Refining the play scala actions
category: scala
tags: scala functional-programming play

---

[Play scala](https://www.playframework.com/) is a web architecture that lets a scala developer to write REST APIs quite easily. One sees a lot of tutorials and samples that implement the actions in somewhat this form

{% highlight scala linenos %}
def normal: Action[AnyContent] = Action.async { implicit request: Request[AnyContent] =>
    Location.locationForm.bindFromRequest().fold(
      errors => Future.successful(BadRequest(errors.errorsAsJson)),
      success => for {
        either <- hogwartsService.fetchHorcrux(success)
      } yield {
        either match {
          case Left(e: OpsFailed) => InternalServerError(Json.toJson(e))
          case Right(horcrux) => Ok(Json.toJson(horcrux))
        }
      }
    )
}
{% endhighlight %}

I feel that this sample contains a lot of noise. If we look closely, the crux of the work being done here is revolving aroud the following lines

{% highlight scala linenos %}
hogwartsService.fetchHorcrux(success)
Ok(Json.toJson(horcrux))
{% endhighlight %}

where we interacting with the domain service layer and then sending back the result from there. The rest of the code is mostly about a common pattern which binds the json body with the model object and proceeds further or fails depending upon whether request is valid or not.

Why not take away all this common work and refine the action definition to its bare necessities. For this, of course, we put this boilerplate code in a common function. One approach we could follow is like this

{% highlight scala linenos %}
def asyncFormAction[T](form: Form[T])(api: (T, CommonHeaders.type) => Future[Result]): Action[AnyContent] = async { implicit request: Request[AnyContent] =>
   form.bindFromRequest().fold({
      (errors: Form[T]) => Future.successful(handleErrors(errors))
    }, {
      (requestObj: T) => api(requestObj, CommonHeaders)
    })
}

private def handleErrors[T](errors: Form[T]): Result = BadRequest(errors.errorsAsJson)
{% endhighlight %}

With the above code, our action defintion in the controller is shortened to this

{% highlight scala linenos %}
def refined: Action[AnyContent] = asyncFormAction(Location.locationForm)((obj: Location, ch: CommonHeaders.type) => for {
    either <- hogwartsService.fetchHorcrux(obj)
  } yield toJsonResult(either)
)
{% endhighlight %}

which only is about the crux of the work we are doing in the controller api. Another advantage of extracting out the form validation and initial request handling to a common function is that one can always also add the logic to extract some common headers related to domain. This is especially useful when one is using [JWT](https://jwt.io/) or similiar technolgies for validations of the requests.

It is also possible to compose the actions in case we need to do a specific request handling like extracting a special header for one-off api.

Feel free to explore the full code in the [github repository](https://github.com/abbi-gaurav/ReactiveMongoPlay) and the controller [HarryPotter.scala](https://github.com/abbi-gaurav/ReactiveMongoPlay/blob/master/app/com/gabbi/controllers/HarryPotter.scala)
