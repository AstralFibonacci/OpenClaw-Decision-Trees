---
id: image-assessment
trigger: "Phase 2 — when evaluating each supplier image for categorization"
---

## Inputs

- **image_url** — URL of the image being evaluated
- **image_index** — position in the image set (1-based)
- **total_images** — total count of images in the set

## Tree

START → check_image_quality

check_image_quality:
  is_blurry == true → EXIT: category_d
  is_duplicate == true → EXIT: category_d
  has_wrong_variant == true → EXIT: category_c
  has_bad_angle == true → EXIT: category_c
  has_text_overlay == true → check_photo_quality
  default → EXIT: category_a

check_photo_quality:
  photo_quality == good → EXIT: category_b
  photo_quality == poor → EXIT: category_d

## Exits

**category_a**
- Action: Tag image as KEEP_AS_IS
- Report: "Image {image_index}: Category A — clean, no action needed"
- Next: Return to skill, process next image or proceed to count_categories

**category_b**
- Action: Tag image as FLAG_FOR_CLEANING
- Report: "Image {image_index}: Category B — good photo, text overlay needs removal"
- Next: Return to skill, process next image or proceed to count_categories

**category_c**
- Action: Tag image as REFERENCE_ONLY
- Report: "Image {image_index}: Category C — wrong variant or bad angle, keep as reference"
- Next: Return to skill, process next image or proceed to count_categories

**category_d**
- Action: Tag image as MARK_FOR_DELETION
- Report: "Image {image_index}: Category D — blurry, duplicate, or poor quality"
- Next: Return to skill, process next image or proceed to count_categories
