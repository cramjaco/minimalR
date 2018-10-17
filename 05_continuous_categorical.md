---
layout: lesson
title: "Session 5: Analyzing continuous data across discrete categories"
output: markdown_document
---



## Learning goals
* Bar plots
* Error bars
* Positioning with jitter and dodging
* Strip charts
* Box plots
* Violin plots
* Factors



## Comparing continuous data
In the last lesson we saw how we could combine various functions from `dplyr` to create summary tables. Since tables can be a bit hard on the eyes, it might be nice to actually plot those data. In this lesson we will look at various tools we can use to represent continuous data across different categories. Let's go ahead and get our `meta_alpha` loaded for this lesson.


```r
library(tidyverse)
library(readxl)

metadata <- read_excel(path="raw_data/baxter.metadata.xlsx",
		col_types=c(sample = "text", fit_result = "numeric", Site = "text", Dx_Bin = "text",
				dx = "text", Hx_Prev = "logical", Hx_of_Polyps = "logical", Age = "numeric",
				Gender = "text", Smoke = "logical", Diabetic = "logical", Hx_Fam_CRC = "logical",
				Height = "numeric", Weight = "numeric", NSAID = "logical", Diabetes_Med = "logical",
				stage = "text")
	)
metadata[["Height"]] <- na_if(metadata[["Height"]], 0)
metadata[["Weight"]] <- na_if(metadata[["Weight"]], 0)
metadata[["Site"]] <- recode(.x=metadata[["Site"]], "U of Michigan"="U Michigan")
metadata[["Dx_Bin"]] <- recode(.x=metadata[["Dx_Bin"]], "Cancer."="Cancer")
metadata[["Gender"]] <- recode(.x=metadata[["Gender"]], "m"="male")
metadata[["Gender"]] <- recode(.x=metadata[["Gender"]], "f"="female")

metadata <- rename_all(.tbl=metadata, .funs=tolower)
metadata <- rename(.data=metadata,
		previous_history=hx_prev,
		history_of_polyps=hx_of_polyps,
		family_history_of_crc=hx_fam_crc,
		diagnosis_bin=dx_bin,
		diagnosis=dx,
		sex=gender)

alpha <- read_tsv(file="raw_data/baxter.groups.ave-std.summary",
		col_types=cols(group = col_character())) %>%
	filter(method=='ave') %>%
	select(group, sobs, shannon, invsimpson, coverage)

meta_alpha <- inner_join(metadata, alpha, by=c('sample'='group'))
```


## Bar Plots
Bar plots are an attractive and popular way to display continuous data that has been divided into categories. For example, we might want to plot the mean FIT Result for each diagnosis group or each obesity category. We may want to get a bit more complicated and plot the mean FIT result value for each combination of diagnosis group and sex. Instead of using `geom_point`, we will use `geom_col` to build bar plots.

Let's start by building a data frame that contains the mean FIT result for each diagnosis group:


```r
fit_by_diagnosis <- meta_alpha %>%
		group_by(diagnosis) %>%
		summarize(mean_fit = mean(fit_result), sd_fit = sd(fit_result))
```

For the `geom_col` function, we can use the `x` (i.e. position on x-axis), `y` (i.e. position on the y-axis), `fill` (i.e. color of the bar itself), `alpha` (i.e transparency of the fill color), `color` (i.e. color of the line that surrounds each bar), `linetype` (i.e. amount of hashing of the line that surrounds each bar), and `size` (i.e. thickness of the surrounding line) aesthetics. Let's start simple...


```r
ggplot(fit_by_diagnosis, aes(x=diagnosis, y=mean_fit)) +
	geom_col()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-3-1.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" width="504" />

Let's change the `fill` color to be `dodgerblue` and set the `alpha` to be `0.5`.


```r
ggplot(fit_by_diagnosis, aes(x=diagnosis, y=mean_fit)) +
	geom_col(fill="dodgerblue", alpha=0.5)
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-4-1.png" title="plot of chunk unnamed-chunk-4" alt="plot of chunk unnamed-chunk-4" width="504" />

To keep the style consistent across our plots, we will end up dumping the `fill` and `alpha` values for now and we will bring back some of the code we used in the earlier plots For now I'm going to copy and paste in the code from the last plot we made


```r
ggplot(fit_by_diagnosis, aes(x=diagnosis, y=mean_fit)) +
	geom_col() +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x="Diagnosis",
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-5-1.png" title="plot of chunk unnamed-chunk-5" alt="plot of chunk unnamed-chunk-5" width="504" />

Clearly there's  a lot wrong with this! First, our bars aren't the correct color. Second, the labels are all wrong. To fix the first problem, we can use `scale_fill_manual` (see what I did there?) in place of `scale_color_manual`. Remember that we need to tell `ggplot` to map the values in the "diagnosis" column to `fill`. We can also clean up the labels to fit this new application - for these types of plots, I would likely leave off the x-axis label.


```r
ggplot(fit_by_diagnosis, aes(x=diagnosis, y=mean_fit, fill=diagnosis)) +
	geom_col() +
	scale_fill_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-6-1.png" title="plot of chunk unnamed-chunk-6" alt="plot of chunk unnamed-chunk-6" width="504" />

We're gaining on it, but the x-axis labels aren't formatted correctly and the order isn't quite right. To fix this we can use the `scale_x_discrete` function with the `limits` and `labels` arguments.


```r
ggplot(fit_by_diagnosis, aes(x=diagnosis, y=mean_fit, fill=diagnosis)) +
	geom_col() +
	scale_fill_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-7-1.png" title="plot of chunk unnamed-chunk-7" alt="plot of chunk unnamed-chunk-7" width="504" />

Now it's properly formatted, but for this plot, we really don't need the legend since it's a bit redundant. We can achieve this by adding `show.legend=FALSE` as an argument to `geom_col`.


```r
ggplot(fit_by_diagnosis, aes(x=diagnosis, y=mean_fit, fill=diagnosis)) +
	geom_col(show.legend=FALSE) +
	scale_fill_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-8-1.png" title="plot of chunk unnamed-chunk-8" alt="plot of chunk unnamed-chunk-8" width="504" />

Awesome. Now let's figure out how we can alter this code to plot the mean FIT result by diagnosis group and sex. Note that if we group by "sex" and "diagnosis", then we need to add an `ungroup` function call to the end of our pipeline to remove the grouping. After creating `fit_by_diagnosis_sex`, we'll map our `fill` aesthetic to the "sex" variable and we'll alter the `scale_fill_manual` syntax to color the bars for females and males differently. Next, we'll bring back in the legend and we'll add a `position="dodge"` argument to `geom_col`. The "dodge" will bump the bars to the side so they don't stack on top of each other.


```r
fit_by_diagnosis_sex <- meta_alpha %>%
		group_by(sex, diagnosis) %>%
		summarize(mean_fit = mean(fit_result), sd_fit = sd(fit_result)) %>%
		ungroup()

ggplot(fit_by_diagnosis_sex, aes(x=diagnosis, y=mean_fit, fill=sex)) +
	geom_col(position=position_dodge()) +
	scale_fill_manual(name=NULL,
		values=c("lightgreen", "orange"),
		breaks=c("female", "male"),
		labels=c("Female", "Male")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-9-1.png" title="plot of chunk unnamed-chunk-9" alt="plot of chunk unnamed-chunk-9" width="504" />

---

### Activity 1
Modify the code we just used to make a grouped bar chart that plots the mean Shannon diversity index by diagnosis group and sex. Group the bars by sex rather than by diagnosis group. Don't worry about the bars being in the correct order for now. See if you can lengthen the y-axis to go from zero to five by using the `coord_cartesian` function

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
shannon_by_diagnosis_sex <- meta_alpha %>%
		group_by(diagnosis, sex) %>%
		summarize(mean_shannon = mean(shannon), sd_fit = sd(shannon)) %>%
		ungroup()

ggplot(shannon_by_diagnosis_sex, aes(x=sex, y=mean_shannon, fill=diagnosis)) +
	geom_col(position=position_dodge()) +
	scale_fill_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("female", "male"),
		labels=c("Female", "Male")) +
	coord_cartesian(ylim=c(0,5)) +
	labs(title="Relationship between diagnosis group, sex, and diversity",
		x=NULL,
		y="Shannon Diversity Index") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-10-1.png" title="plot of chunk unnamed-chunk-10" alt="plot of chunk unnamed-chunk-10" width="504" />
</div>

---

One last thing with bar plots is to represent the variation in the data with error bars. We can do this with the `geom_errorbar`. This geoms will take `x`, `y`, `ymax`, `ymin`, `alpha`, `color`, `linetype`, `size`, `width` as aesthetics. The two that we haven't seen before are `ymax` and `ymin`, which you can probably deduce from their names refer to where the top and bottom lines should be drawn on the error bars, respectively. To try this out, let's return to our `fit_by_diagnosis` data frame


```r
ggplot(fit_by_diagnosis, aes(x=diagnosis, y=mean_fit, fill=diagnosis)) +
	geom_col(show.legend=FALSE) +
	geom_errorbar(aes(ymax=mean_fit+sd_fit, ymin=mean_fit-sd_fit)) +
	scale_fill_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-11-1.png" title="plot of chunk unnamed-chunk-11" alt="plot of chunk unnamed-chunk-11" width="504" />

Cool. Well. Kind of. There are two problems, one far more significant than the other. First, the standard deviations on our means are huge. Also, considering the bottom error bar goes below zero, it's safe to assume that our data probably aren't normally distributed. This is kind of a big problem with bar plots that makes them a [less than desirably tool](http://journals.plos.org/plosbiology/article?id=10.1371/journal.pbio.1002128) for presenting mean data. The other problem is that the width of the error bars is as wide as the bars themselves. Let's change them to be half the width of the rectangles. We can do this by setting the `width` aesthetic in geom_errorbar to `0.5`.


```r
ggplot(fit_by_diagnosis, aes(x=diagnosis, y=mean_fit, fill=diagnosis)) +
	geom_col(show.legend=FALSE) +
	geom_errorbar(width=0.5, aes(ymax=mean_fit+sd_fit, ymin=mean_fit-sd_fit)) +
	scale_fill_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-12-1.png" title="plot of chunk unnamed-chunk-12" alt="plot of chunk unnamed-chunk-12" width="504" />

One thing we could do is to set the `ymin` value to `mean_fit - 0.1 * mean_fit` rather than `mean_fit-sd_fit`. But this will put a black line just inside the black rectangle. We can hide the bottom error bar by calling `geom_col` after `geom_errorbar`


```r
ggplot(fit_by_diagnosis, aes(x=diagnosis, y=mean_fit, fill=diagnosis)) +
	geom_errorbar(width=0.5, aes(ymax=mean_fit+sd_fit, ymin=mean_fit-0.1*mean_fit)) +
	geom_col(show.legend=FALSE) +
	scale_fill_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-13-1.png" title="plot of chunk unnamed-chunk-13" alt="plot of chunk unnamed-chunk-13" width="504" />

Better. This is kind of a ridiculous example, but it helps to underscore the problems with bar plots. Let's look at a few other ways to represent continuous data from multiple categories. We'll find that much of the syntax we used with `geom_col` is the same for these other `geom`s.


---

### Activity 2
Create a bar plot with error bars that shows the Shannon diversity for each diagnosis group grouped by sex along the x-axis. Add error bars representing the standard deviations to the bar plots.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
shannon_by_diagnosis_sex <- meta_alpha %>%
		group_by(diagnosis, sex) %>%
		summarize(mean_shannon = mean(shannon), sd_shannon = sd(shannon)) %>%
		ungroup()

ggplot(shannon_by_diagnosis_sex, aes(x=sex, y=mean_shannon, fill=diagnosis)) +
	geom_errorbar(position=position_dodge(width=0.9), width=0.5, aes(ymax=mean_shannon+sd_shannon, ymin=mean_shannon-sd_shannon)) +
	geom_col(position=position_dodge()) +
	scale_fill_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("female", "male"),
		labels=c("Female", "Male")) +
	labs(title="Relationship between diagnosis group, sex, and diversity",
		x=NULL,
		y="Shannon Diversity Index") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-14-1.png" title="plot of chunk unnamed-chunk-14" alt="plot of chunk unnamed-chunk-14" width="504" />
</div>

---


## Strip charts
As I mentioned above, one limitation of bar plots is that they obscure the data and make it hard to determine whether the data are normally distributed and make it unclear how many observations are within each bar. An alternative to this is a strip chart where the y-axis values are plotted for each observation in the category. We can make these plots in `ggplot` using `geom_jitter`. Also, notice that we no longer want the data frame with the summary statistics, instead we can use the full `meta_alpha` data frame.


```r
ggplot(meta_alpha, aes(x=diagnosis, y=fit_result, color=diagnosis)) +
	geom_jitter(pch=19, size=2) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-15-1.png" title="plot of chunk unnamed-chunk-15" alt="plot of chunk unnamed-chunk-15" width="504" />

Looks better, eh? Perhaps we'd like to alter the jitter along the x-axis, we can set the amount of jitter using the `width` argument


```r
ggplot(meta_alpha, aes(x=diagnosis, y=fit_result, color=diagnosis)) +
	geom_jitter(pch=19, size=2, width=0.2) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-16-1.png" title="plot of chunk unnamed-chunk-16" alt="plot of chunk unnamed-chunk-16" width="504" />

---

### Activity 3
Create a strip chart that shows the Shannon diversity for each diagnosis category. You'll notice that the y-axis does not seem to extend down to zero. Use the `coord_cartesian` function to include zero.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
ggplot(meta_alpha, aes(x=diagnosis, y=shannon, color=diagnosis)) +
	geom_jitter(pch=19, size=2, width=0.2) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between Shannon diversity and subject's diagnosis",
		x=NULL,
		y="Shannon Diversity Index") +
	theme_classic() +
	coord_cartesian(ylim=c(0,5))
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-17-1.png" title="plot of chunk unnamed-chunk-17" alt="plot of chunk unnamed-chunk-17" width="504" />
</div>

---

Great. How about comparing FIT results for combinations of sex and diagnosis? One option is to place the points along the x-axis by sex and color by diagnosis.


```r
ggplot(meta_alpha, aes(x=sex, y=fit_result, color=diagnosis)) +
	geom_jitter(pch=19, size=2, width=0.2) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("female", "male"),
		labels=c("Female", "Male")) +
	labs(title="Relationship between FIT result and subject's diagnosis and sex",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-18-1.png" title="plot of chunk unnamed-chunk-18" alt="plot of chunk unnamed-chunk-18" width="504" />

Meh. Another option is to use a the `position_jitterdodge` function with the `position` argument in `geom_jitter`. Recall that "dodge" means move the points so they don't overlap and "jitter" randomizes the x-position of the points (you can also randomize them on the y-axis, but that seems... strange).


```r
ggplot(meta_alpha, aes(x=sex, y=fit_result, color=diagnosis)) +
	geom_jitter(pch=19, size=2, position=position_jitterdodge()) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("female", "male"),
		labels=c("Female", "Male")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-19-1.png" title="plot of chunk unnamed-chunk-19" alt="plot of chunk unnamed-chunk-19" width="504" />

It looks like our diagnosis groups are still overlapping a bit. We can give a jitter.width and dodge.width value to `position_jitter_dodge` to eliminate that overlap.


```r
ggplot(meta_alpha, aes(x=sex, y=fit_result, color=diagnosis)) +
	geom_jitter(pch=19, size=2, position=position_jitterdodge(dodge.width=0.7, jitter.width=0.2)) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("female", "male"),
		labels=c("Female", "Male")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-20-1.png" title="plot of chunk unnamed-chunk-20" alt="plot of chunk unnamed-chunk-20" width="504" />

The order of the diagnosis groups is still out of whack. We'll come back to that later.


---

### Activity 4
Create a strip chart that shows the Shannon diversity for each diagnosis category and sex. Again, make sure that the y-axis includes zero.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
ggplot(meta_alpha, aes(x=sex, y=shannon, color=diagnosis)) +
	geom_jitter(pch=19, size=2, position=position_jitterdodge(dodge.width=0.7, jitter.width=0.2)) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("female", "male"),
		labels=c("Female", "Male")) +
	labs(title="Relationship between Shannon diversity and subject's diagnosis and sex",
		x=NULL,
		y="Shannon Diversity Index") +
	theme_classic() +
	coord_cartesian(ylim=c(0,5))
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-21-1.png" title="plot of chunk unnamed-chunk-21" alt="plot of chunk unnamed-chunk-21" width="504" />
</div>

---

## Box plots
I like strip charts because I can see all of the data. These get a bit messy when there are a large number of observations. They are also problematic because although they show all of the data, we aren't great at identifying the median or the intraquartile ranges. An alternative to the strip chart that solves these problems is the box plot. That being said, a box plot may not be meaningful if there aren't many observations. We can generate a box plot using the `geom_boxplot` function in much the same way we did earlier with the `geom_jitter`


```r
ggplot(meta_alpha, aes(x=diagnosis, y=fit_result, color=diagnosis)) +
	geom_boxplot() +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-22-1.png" title="plot of chunk unnamed-chunk-22" alt="plot of chunk unnamed-chunk-22" width="504" />

One of the things I don't like about box plots is that it isn't always clear what the various parts of the box or whiskers represent. The line through the middle of the rectangle is the median value and the lower and upper edges of the rectangle represent the 25th and 75th percentiles. The whiskers extend to the larges value greater than 1.5 times the difference between the 25th and 75th percentiles. It's a way to represent outliers. Another way to represent the distribution is wiht a notched box plot


```r
ggplot(meta_alpha, aes(x=diagnosis, y=fit_result, color=diagnosis)) +
	geom_boxplot(notch=TRUE) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-23-1.png" title="plot of chunk unnamed-chunk-23" alt="plot of chunk unnamed-chunk-23" width="504" />

In this case, the notches extend to 1.58 times the difference between the 25th and 75th percentiles divided by the square root of the number of observations. According to `?geom_boxplot` this gives a sense of the 95% confidence interval for comparing medians and "if the notches of two boxes do not overlap, this suggests that the medians are significantly different". Alternatively, you could generate an ugly and busy plot (but people seem to like them) where a strip chart and box plot (without the outliers) are overlapped using the `outlier.color="white"` argument in `geom_boxplot` (the `outlier.color` value should match your background)


```r
ggplot(meta_alpha, aes(x=diagnosis, y=fit_result, color=diagnosis)) +
	geom_boxplot(outlier.color="white", pch=0) +
	geom_jitter(pch=19, position=position_jitterdodge(jitter.width=0.75)) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-24-1.png" title="plot of chunk unnamed-chunk-24" alt="plot of chunk unnamed-chunk-24" width="504" />


---

### Activity 5
Make a box plot that shows the Shannon diversity for each sex grouped by the subjects' diagnosis. Make the same plot, but group by diagnosis. Which is better? When would you want to group by sex? By diagnosis?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
ggplot(meta_alpha, aes(x=diagnosis, y=shannon, color=sex)) +
	geom_boxplot() +
	scale_color_manual(name=NULL,
		values=c("lightgreen", "orange"),
		breaks=c("female", "male"),
		labels=c("Female", "Male")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between Shannon diversity and subject's sex and diagnosis",
		x=NULL,
		y="Shannon diversity index") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-25-1.png" title="plot of chunk unnamed-chunk-25" alt="plot of chunk unnamed-chunk-25" width="504" />


```r
ggplot(meta_alpha, aes(x=sex, y=shannon, color=diagnosis)) +
	geom_boxplot() +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("female", "male"),
		labels=c("Female", "Male")) +
	labs(title="Relationship between Shannon diversity and subject's sex and diagnosis",
		x=NULL,
		y="Shannon diversity index") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-26-1.png" title="plot of chunk unnamed-chunk-26" alt="plot of chunk unnamed-chunk-26" width="504" />
It depends on the question, which is better! If we are interested in comparing the two sexes, then we want to group by sex. If we want to compare the diagnosis groups, then we'll want to group by diagnosis groups.
</div>

---

### Activity 6
Our box plots have only had color on the rectangle, median line, whiskers, and outliers. Generate a box plot for the relationship between the patients' Shannon diversity and their diagnosis. Add a complimentary fill color that allows you to still see the cardinal values of the box plot.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
ggplot(meta_alpha, aes(x=diagnosis, y=shannon, color=diagnosis, fill=diagnosis)) +
	geom_boxplot() +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_fill_manual(name=NULL,
		values=c("lightblue", "pink", "gray"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between Shannon diversity and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-27-1.png" title="plot of chunk unnamed-chunk-27" alt="plot of chunk unnamed-chunk-27" width="504" />
</div>

---

## Violin plots
In the last box plot example, we plotted the data points on top of the box plot. This is pretty cluttered and ugly. An alternative is the violin plot, where the position along the left axis indicates the density of values at that position on the y-axis. You can create violin plots very much in the same way as strip carts and box plots using the `geom_violin`


```r
ggplot(meta_alpha, aes(x=diagnosis, y=fit_result, fill=diagnosis)) +
	geom_violin() +
	scale_fill_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-28-1.png" title="plot of chunk unnamed-chunk-28" alt="plot of chunk unnamed-chunk-28" width="504" />

---

### Activity 7
In the previous violin plot we created the outline color to the violins was black. Can you get the outline color to match that of the fill color?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
ggplot(meta_alpha, aes(x=diagnosis, y=fit_result, fill=diagnosis, color=diagnosis)) +
	geom_violin() +
	scale_fill_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-29-1.png" title="plot of chunk unnamed-chunk-29" alt="plot of chunk unnamed-chunk-29" width="504" />
</div>

---

### Activity 8
Create a violin plot comparing diversity across diagnosis groups and sex

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
ggplot(meta_alpha, aes(x=sex, y=shannon, fill=diagnosis, color=diagnosis)) +
	geom_violin() +
	scale_fill_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("female", "male"),
		labels=c("Female", "Male")) +
	labs(title="Relationship between Shannon diversity and subject's diagnosis and sex",
		x=NULL,
		y="Shannon diversity index") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-30-1.png" title="plot of chunk unnamed-chunk-30" alt="plot of chunk unnamed-chunk-30" width="504" />
</div>

---

### Activity 9
A new variant of the types of plots discussed in this lesson is the ridgeline plot (aka ["joy plot"](http://www.houstonpress.com/music/five-joy-division-covers-that-dont-suck-6518316)). Install the `ggridges` package and see if you can figure out how to build a ridgeline plot of Shannon diversity values for the three diagnosis groups.

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">

```r
#install.packages("ggridges")
library(ggridges)
```

```
## Error in library(ggridges): there is no package called 'ggridges'
```

```r
ggplot(meta_alpha, aes(x=shannon, y=diagnosis, color=diagnosis, fill=diagnosis)) +
	geom_density_ridges(alpha=0.5) +
	scale_fill_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_y_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between Shannon diversity and subject's diagnosis",
		y=NULL,
		x="Shannon diversity index") +
	theme_classic()
```

```
## Error in geom_density_ridges(alpha = 0.5): could not find function "geom_density_ridges"
```
</div>

---

## Ordering our groups
Whenever we've grouped our data by sex, the diagnosis groups are ordered alphabetically within each group (i.e. Adenoma, Cancer, Normal) rather than in our desired order of disease progression (i.e. Normal, Adenoma, Cancer). To fix the ordering, we need to cast these variables as factors. Factors are a troublesome feature within R. Thankfully, there's the `forcats` package within the tidyverse, which makes working with factors much easier. Factors are a type of data for representing categorical data. Characters are another type of data for representing categorical data, but the categories are ordered alphabetically. Sometimes we want to order them in another way. For example, if we have a column that has months, then when we plot with month on the x-axis, "April" will come first rather than "January". We can also rename factors so that "jan" is displayed as "January".  We've kind of already seen this when we relabeled "normal" with "Normal".

Let's return to the example of generating the bar plot of plotting the FIT result grouped by sex and then by diagnosis group.


```r
ggplot(meta_alpha, aes(x=sex, y=fit_result, color=diagnosis)) +
	geom_jitter(pch=19, size=2, position=position_jitterdodge(dodge.width=0.7, jitter.width=0.2)) +
	scale_color_manual(name=NULL,
		values=c("blue", "red", "black"),
		breaks=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	scale_x_discrete(limits=c("female", "male"),
		labels=c("Female", "Male")) +
	labs(title="Relationship between FIT result and subject's diagnosis",
		x=NULL,
		y="FIT Result") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-32-1.png" title="plot of chunk unnamed-chunk-32" alt="plot of chunk unnamed-chunk-32" width="504" />

We can reorder the diagnosis variable by using the `factor` function where we give it the levels for the factor in the order we want it in. You might notice that we previously used a bit of a hack to set the `values` argument in `scale_color_manual`. This argument was taking our diagnosis values in alphabetical order. The values for `breaks` and `labels` were the order we wanted. Now we can use the "correct" order for our `values` argument


```r
meta_alpha %>%
	mutate(diagnosis = factor(diagnosis, levels=c("normal", "adenoma", "cancer"))) %>%
	ggplot(aes(x=sex, y=fit_result, color=diagnosis)) +
		geom_jitter(pch=19, size=2, position=position_jitterdodge(dodge.width=0.7, jitter.width=0.2)) +
		scale_color_manual(name=NULL,
			values=c("black", "blue", "red"),
			breaks=c("normal", "adenoma", "cancer"),
			labels=c("Normal", "Adenoma", "Cancer")) +
		scale_x_discrete(limits=c("female", "male"),
			labels=c("Female", "Male")) +
		labs(title="Relationship between FIT result and subject's diagnosis",
			x=NULL,
			y="FIT Result") +
		theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-33-1.png" title="plot of chunk unnamed-chunk-33" alt="plot of chunk unnamed-chunk-33" width="504" />

Nice, eh? There are a variety of things you can do with factors including reordering the factors by another variable, aggregating multiple values, and renaming variables. These are really outside the scope of this tutorial and I rarely use them in my work. You can learn more about them in the [R4DS book](http://r4ds.had.co.nz/factors.html).

---

### Activity 10
Generate a box plot clustering Shannon diversity data by diagnosis and then sex. Have the strips for males go before the females. Why didn't we have to worry about factors before with the sex variable?

<input type="button" class="hideshow">
<div markdown="1" style="display:none;">
This wasn't an issue before because we plotted females before males, which is alphabetical order.


```r
meta_alpha %>%
	mutate(sex = factor(sex, levels=c("male", "female"))) %>%
ggplot(aes(x=diagnosis, y=shannon, color=sex)) +
	geom_boxplot() +
	scale_color_manual(name=NULL,
		values=c("orange", "lightgreen"),
		breaks=c("male", "female"),
		labels=c("Male", "Female")) +
	scale_x_discrete(limits=c("normal", "adenoma", "cancer"),
		labels=c("Normal", "Adenoma", "Cancer")) +
	labs(title="Relationship between Shannon diversity and subject's sex and diagnosis",
		x=NULL,
		y="Shannon diversity index") +
	theme_classic()
```

<img src="assets/images/05_continuous_categorical//unnamed-chunk-34-1.png" title="plot of chunk unnamed-chunk-34" alt="plot of chunk unnamed-chunk-34" width="504" />
</div>
