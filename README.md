# Understanding how the image is built using conditioningZeroOut and prompt injection

To better understand prompt injection, it is important to know what happens without prompt injection. This allows us to better measure the changes we are achieving.

<h2> The model and the prompt </h2>

ConditioningZeroOut is supposed to ignore the prompt no matter what is written. So you'd expect to get no images. But you do get images.  

![ZeroConditionning](https://github.com/Creative-comfyUI/prompt_injection/assets/166729777/8dff331e-e087-4010-bc21-52335d55419c)

Either the model passes instructions when there is no prompt, or ConditioningZeroOut doesn't work and zero doesn't mean zero.

Let try the model withou the clip. In the ksampler we don't link the model with the checkpoint loader, but with the unet loader (same model). The Unet Loader is the model in its raw state without taking the clip into account. Again there is no change. In other words, the ksampler is still receiving information.

Let's vary the seed to see if the result changes, which would be logical. We have a new set of images without knowing where they came from, except that they are the images used to build the model.

This means that in the absence of a clip, the model gives instructions to the Ksampler. To test my hypothesis, let's not link any model to the Ksampler. The way to do this is to cancel the model by subtracting it from the same model to get 0 model. In this case we only have noise. 

<b> Without the prompt, the model defines random images. These are used as a reference.</b>  

<h2>Now let's look at the prompt injection </h2>

<b>IMPORTANT INFORMATION : These examples are made with models trained with few steps as Turbo. Regular models don't give such good results. .</b>

![workflow-6](https://github.com/Creative-comfyUI/prompt_injection/assets/166729777/611cc6cb-7980-4ce1-91b5-07a4920749b0)


We now know that for a given seed we get a series of images without any prompt. In my previous analysis (https://github.com/Creative-comfyUI/seed_eng) I showed that if we use Ksampler (inspire) or Ksampler with noise injection we get the same images for a batch size of 3 with Euler. So, to avoid variations, we'll use Euler with Ksampler inspire to test the prompt injection. 

Here is our basic image without prompt with Euler. These images are defined randomly by the model. 

![Screenshot 2024-06-08 at 1 24 46 PM](https://github.com/Creative-comfyUI/prompt_injection/assets/166729777/0c172765-583f-4260-9f27-c37671ea1f40)

The prompt injection 4, 5 , 7 always gives the same basic images no change 

The prompt injection input 8 only brings changes to images 2 and 3, but not to image 1, which is surprising because the Euler sampler doesn't bring any variation. This means that block 8 only affects the image following the first image, as if digital noise had been added. If I change the batch size from 3 to 1, the image that appears is not the first image, as you might expect, but the image that matches the prompt.  This means that from block 8 the changes take place and image 1, which never changes, is a reference image. 

![Screenshot 2024-06-08 at 2 02 00 PM](https://github.com/Creative-comfyUI/prompt_injection/assets/166729777/e8366505-da4f-4e57-89ff-e461ef79e179)

If we now change the word woman in the prompt to man, we get almost the same pose, the same style, although this time we lose the colour.  As you can see, the first image is still the same. 

![Screenshot 2024-06-08 at 2 15 09 PM](https://github.com/Creative-comfyUI/prompt_injection/assets/166729777/3e1744e8-e7f0-4928-b379-6c34b4469701)

Let's change the style in the prompt and see what happens. The image changes, but the style isn't what you'd expect. If you notice, this time picture 2 and 3 are the same. This means that for some words the associations with their references can be different. 

![Screenshot 2024-06-08 at 2 19 47 PM](https://github.com/Creative-comfyUI/prompt_injection/assets/166729777/f29481fb-6858-4e33-ae76-9d3058f7e887)

Let's replace man with woman again to see if the style is more appropriate, or we'll have the same problem. The style is not applied. However, it retains all the atmosphere of the prompt, as do all the previous images. 

![Screenshot 2024-06-08 at 2 25 38 PM](https://github.com/Creative-comfyUI/prompt_injection/assets/166729777/bc854ffb-e256-4bc9-beed-7de93bede84e)

Now let us try different prompt with the same seed 

![output8](https://github.com/Creative-comfyUI/prompt_injection/assets/166729777/5a82a7ee-9f25-46d7-a7b9-1f19cd573733)

As you can see, whatever I've written, the position is the same, even if I didn't mention it in the prompt. 
 
This means that block 8 doesn't focus on colour or style, but on subject. The variation concerns the character, but not the pose or angle of view, which remain the same. <b> The angle of view is determined by the format of the latent, not by what is written </b>. By changing the format, the camera change it is point of view, but the atmosphere remains the same. Surprisingly, the first image is not the same at all, while 1 and 2 still correspond to what is written. 

![Latentformat](https://github.com/Creative-comfyUI/prompt_injection/assets/166729777/c572a68e-3337-4e06-9ba8-04ccb5d8b90f)


Midle block hasn't made any changes either. Image 2 et 3 is quite the same of image 1, apart from a slight variation in the dress. Once again, model 1 remains unchanged,the reference model

Block output 0 affects the face, hair and clothes, but in a light way, like Middle. 

With block output 1, we can start to see a better match with our prompt. We realise that we could stop here and the picture would be perfect. But let's continue 

![Screenshot 2024-06-08 at 3 32 57 PM](https://github.com/Creative-comfyUI/prompt_injection/assets/166729777/65199a19-3574-444e-b408-6d2289e55fd2)

If you look carefully, there are many similarities between the two pictures, with the differences that the prompt has been applied to pictures 2 and 3. 
For example: hair, bare shoulder, decoration in the hair, ear loop. All this information was not mentioned in the prompt!    

The block output 2, 3, 4, 5 gives the same image as image number 1 with slight variations. We're back where we started.

Let's apply the prompt as we have the habit to apply, keeping block 5.  In principle, we should get the image we would have had if we hadn't used the prompt injection. And that image corresponds to the ambience and caracter position of block input 8 but with the finishing of the image of block output 1.

![Screenshot 2024-06-08 at 3 52 24 PM](https://github.com/Creative-comfyUI/prompt_injection/assets/166729777/6fb17770-8bb6-48a6-8a95-ea5aa946dd3f)

Now let's render based on output 1 to see what happens if we use the prompt as we have the habit to apply. Nothing happens and we get the image we would normally have obtained.

Our prompt is not the only reference taken into account. The model proposes a reference image whose noise contributes to the noise generated by our prompt. Therefore, when we use clip vision to compose an image from two images, we find elements belonging to this reference image. 

Based on output 1, I changed seed and mute the prompt with zeroConditionning. We see exactly what we have seen previously. Images 2 and 3 are based on the first, similarity in the face and the position and even the clothes and thee flowers have been replaced by a kind of crown. In fact, if you pay attention, the prompt has been applied to image 1 to give 2 and 3. 

![Screenshot 2024-06-08 at 4 20 21 PM](https://github.com/Creative-comfyUI/prompt_injection/assets/166729777/fc21561a-bc8c-4229-a59c-24845b7fa513)

This means that there is a reference image whose noise is used to generate the final image base on the clip (the prompt we wrote). This reference image is probably the one that the clip vision retrieves when an image is submitted. It wouldn't just use the image we see on the screen, but the image reference is used to construct the new image. 

Furthermore, we can assume that in the Usenet model the real change occurred at block input 8 and block output 1. Certainly, when we play with noise and weight, many things can change.

In my opinion prompt injection offers new possibilities to play with image and modify it with noise before the final result. More over using prompt injection can let us give another look to image in stopping before the process and complete with another prompt or model. I am sure that combination can surprise us. This is the aim of my work seen I start comfyUI using all kind of composition to create art. (https://www.artmajeur.com/harry-fabre)

In part 2 I will make more variations and use Ipadapter.

  
