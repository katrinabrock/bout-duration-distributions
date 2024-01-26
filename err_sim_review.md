# Big Picture Topics


## Code Quality

### Optimize for Readability

Here, I don't expect you to make any changes to the code (unless you are doing a huge refactor for other reason), this feedback is more to keep in mind the next time you embark on a similar project.

In my mind, optimizing for readability would mean having the processes in the code mirror as closely as possible how researchers conceptualize the real proccess both in name and structure. It seems like there are a few processes you're simulating here:
1) generating the ground truth behavioral sequences from some process/distribution,
2) the conversion between the truth and some measurable values (you call them "features" here),
3) inferring observed behavioral sequence from measurements
4) inferring the generating process from inferred behavioral states. Within process 2 and 4 there are sub-processes where you convert back and forth between discrete and continuous sequences. I'd also add a step 5 of plotting the simlation results.

Your current code is something like this:
```
/simulate.py
for param_set in step_1_param_space; (params used in step 1 only)
    /simulations.simulate_with_distribution
    set error rate params (used in step 2)
    add some details to ground truth distribution
    for i in 1:nsimlations:
        //simlations._simulate_and_get_results
        set num bouts (param related to step 2 because it's the amount that we observe)
        simulations.simulateor.Simulator
            do step 1
            for param_set in step_2_param_space:
                do step 2 (Deleniation between 1 and 2 not very clean)
        do step 3 (no for loop here because always done the same way)
        do step 4 (no parametrized loop here because always done the same way)
    save summary to disk
step 5 (plotting)
```
For the for loops listed above, the iterations are independent so in theory they can be paralellized and one of them is.
A more readable structure would be something like this
```
universe_of_ground_dists_params = [....] # each item here is an object that fully specifies the variables needed to generate a ground truth sequence, these shouldn't need modification later to be used
universe_of_measurement_params = [...] # should include not just the err params, but also nbouts, epoch, etc other things related to the translation from ground truth to measurement
(opt) state_classification_params = [...] # this is if you want to change fitting.classifier
(opt) dist_inference_params = [...] # e.g if you want to test multiple models 
```

Then there a couple ways that I can think of that you can structure the code itself in readable way:

```
def run_study(a_set_of_ground_dist_params, a_set_of_measurement_params, ...):
    ground_truth_bout_lengths = generate_ground_truth(a_set_of_ground_truth_dist_params)
    measurements (or features) = observe(ground_truth_bout_lengths, measurement_params)
    inferred_states = infer_states(features)
    inferred_distributions = infer_distributions(inferred_states)
    return (infered_distributions)

results = {}
for gtp in universe_of_ground_truth_params:
    for mesp in universe_of_measurment_params:
        for i in range(nsims_per_param_set):
            results[(gtp, mesp, i)] = run_study(gtp, mesp)
plot_results(results)
```

or alternatively

```
all_ground_truth_data = generate_ground_truth(universe_of_ground_dist_params, nsim_per_param_set)
all_measurements = generate_measurements(universe_of_measurement_params, all_gorund_truth_data)
inferred_states = infer_states(all_measurments)
inferred_distributions = infer_distributions(inferred_states)
```

Of course, this isn't to stop you from paralellizing,
you can farm out any of the written (or implied inside a function) for loops into mp.
The point is to make functions/components named based on the real processes the represent.
Break implementation complexity into those components, then push it inside these readably named functions and objects. 
This will let you more easily iterate on each piece independently and also make the flow of data and execution flow more clear to the reader.
For example, when you're tyring to illustrate your specific case, the main thing you're trying to do explore a smaller parameter spece with more granularity in step 2 for one value of step 1, but because the logic is all mixed together between generating the parameter space, and executing the different steps, you had to copy a lot of the code and tweak it here and there instead of changing the in puts and reusing code. (You were able to reuse the some code where parts your were not changing were encapusated.)
    
    
## Conceptual/Design Feedback

### Assumptions
Here are some assumptions that I think may impact the results. IMO it would be good to one of the following with each of them:
* actually if they matter
* explain in the text (at least in the appendix) that there's some logical reason that they wouldn't impact the results
* acknowledge in the text that this is an untested assumption

Here's my list:
* Only two states
* Distribution is the same for both states
* Error rate is the same for both states


## Clarifications


# Smaller Issues

## Must Check

## Other