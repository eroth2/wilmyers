# wilmyers
Wil Myers 2019 vs 2020 Project
#Eric Roth Wil Myers Project

###############################################

#scrape Wil Myer's pitch by pitch data from 2019 & 2020

playerid_lookup(last_name = "Myers", first_name = "Wil") %>% view()
scrape_statcast_savant_batter(start_date = "2019-3-28", end_date = "2019-9-29",
                              batterid = 571976) -> wmyers19
scrape_statcast_savant_batter(start_date = "2020-7-23", end_date = "2020-9-26",
                              batterid = 571976) -> wmyers20

#group all swing and misses together
ifelse(wmyers19$description == "swinging_strike_blocked", "swinging_strike", wmyers19$description) -> wmyers19$description
ifelse(wmyers20$description == "swinging_strike_blocked", "swinging_strike", wmyers20$description) -> wmyers20$description
                              
#Rename pitch_types
ifelse(wmyers19$pitch_type == "KC", "KNCU", wmyers19$pitch_type) -> wmyers19$pitch_type
ifelse(wmyers19$pitch_type == "FC", "CUT", wmyers19$pitch_type) -> wmyers19$pitch_type
ifelse(wmyers19$pitch_type == "FF", "4sm", wmyers19$pitch_type) -> wmyers19$pitch_type
ifelse(wmyers19$pitch_type == "FT", "2sm", wmyers19$pitch_type) -> wmyers19$pitch_type
ifelse(wmyers19$pitch_type == "FS", "SPL", wmyers19$pitch_type) -> wmyers19$pitch_type
ifelse(wmyers19$pitch_type == "SI", "SNK", wmyers19$pitch_type) -> wmyers19$pitch_type

ifelse(wmyers20$pitch_type == "KC", "KNCU", wmyers20$pitch_type) -> wmyers20$pitch_type
ifelse(wmyers20$pitch_type == "FC", "CUT", wmyers20$pitch_type) -> wmyers20$pitch_type
ifelse(wmyers20$pitch_type == "FF", "4sm", wmyers20$pitch_type) -> wmyers20$pitch_type
ifelse(wmyers20$pitch_type == "FS", "SPL", wmyers20$pitch_type) -> wmyers20$pitch_type
ifelse(wmyers20$pitch_type == "SI", "SNK", wmyers20$pitch_type) -> wmyers20$pitch_type

#create "count" column
wmyers19 %>% mutate(count = paste(wmyers19$balls,"-",wmyers19$strikes)) -> wmyers19
wmyers20 %>% mutate(count = paste(wmyers20$balls,"-",wmyers20$strikes)) -> wmyers20

#trim down to just balls put in play and make xwoba numeric        
wmyers19 %>% filter(type == "X") -> wmyers19_bip
wmyers20 %>% filter(type == "X") -> wmyers20_bip

as.numeric(wmyers19$estimated_woba_using_speedangle) -> wmyers19$estimated_woba_using_speedangle
as.numeric(wmyers20$estimated_woba_using_speedangle) -> wmyers20$estimated_woba_using_speedangle

#create launch angle + exit velo with relation to xwoba chart
ggplot(wmyers19_bip, aes(launch_speed, launch_angle)) + geom_point(aes(color = estimated_woba_using_speedangle)) +
  scale_color_gradient(low = "blue", high = "red") + easy_add_legend_title("xWOBA") +
  geom_hline(yintercept = 25, linetype = "dashed") + geom_hline(yintercept = 35, linetype = "dashed") +
  geom_vline(xintercept = 95, linetype = "dashed") + 
  annotate(geom = "text", x = 40, y = 20, label = "25°") +
  annotate(geom = "text", x = 40, y = 40, label = "35°") +
  annotate(geom = "text", x = 101, y = -80, label = "95 mph") + 
  xlab("Exit Velocity (MPH)") + ylab("Launch Angle (Degrees)") +
  labs(title = "2019 Launch Angle, Exit Velo, & xWOBA Relation") + 
  theme(plot.title = element_text(face = "bold", hjust = .5)) +
  xlim(c(40,110)) -> wmyers19_LA_EV
  
 ggplot(wmyers20_bip, aes(launch_speed, launch_angle)) + geom_point(aes(color = estimated_woba_using_speedangle)) +
  scale_color_gradient(low = "blue", high = "red") + easy_add_legend_title("xWOBA")+
  geom_hline(yintercept = 25, linetype = "dashed") + geom_hline(yintercept = 35, linetype = "dashed") +
  geom_vline(xintercept = 95, linetype = "dashed") + 
  annotate(geom = "text", x = 33, y = 20, label = "25°") +
  annotate(geom = "text", x = 33, y = 40, label = "35°") +
  annotate(geom = "text", x = 101, y = -80, label = "95 mph") + 
  xlab("Exit Velocity (MPH)") + ylab("Launch Angle (Degrees)") +
  labs(title = "2020 Launch Angle, Exit Velo, & xWOBA Relation") + 
  theme(plot.title = element_text(face = "bold", hjust = .5)) +
  xlim(c(40,110)) -> wmyers20_LA_EV

#find "sweet spot" % and hard hit%, as well as both combined
nrow(wmyers19_bip %>% filter(launch_angle >= 8 & launch_angle <= 32))/nrow(wmyers19_bip)
nrow(wmyers20_bip %>% filter(launch_angle >= 8 & launch_angle <= 32))/nrow(wmyers20_bip)

nrow(wmyers19_bip %>% filter(launch_speed >= 95))/nrow(wmyers19_bip)
nrow(wmyers20_bip %>% filter(launch_speed >= 95))/nrow(wmyers20_bip)

#find all different pitches Myers has seen and omit "nulls"
unique(wmyers19$pitch_type)
unique(wmyers20$pitch_type)
wmyers19 %>% subset(pitch_type!= "null") -> wmyers19
wmyers20 %>% subset(pitch_type!= "null") -> wmyers20

############################
#VS righties when applicable

#Create pitch percentage by count vs RHP
wmyers19 %>% mutate(year = 2019) -> wmyers19
wmyers20 %>% mutate(year = 2020) -> wmyers20
as.character(wmyers19$year) -> wmyers19$year
as.character(wmyers20$year) -> wmyers20$year

wmyers19_R %>% group_by(pitch_type, year) %>% 
  summarize(n = n(),
            pct = n/nrow(wmyers19_R)*100) %>% select(-n) -> wmyers19_R_pct
wmyers19_R_pct %>% mutate(pct = round(pct, digits = 1)) -> wmyers19_R_pct

wmyers20_R %>% group_by(pitch_type, year) %>% 
  summarize(n = n(),
            pct = n/nrow(wmyers20_R)*100) %>% select(-n) -> wmyers20_R_pct
wmyers20_R_pct %>% mutate(pct = round(pct, digits = 1)) -> wmyers20_R_pct

rbind(wmyers19_R_pct, wmyers20_R_pct) -> wmyers_pct_combinedR

#pitch distribution chart
ggplot(wmyers_pct_combinedR, aes(reorder(pitch_type, -pct), pct, fill = year)) + 
  geom_bar(stat = "identity", position = "dodge2") + 
  scale_fill_manual(values = c("#8B4513", "yellow")) +
  xlab("Pitch Type") + ylab("% Thrown")+
  geom_text(aes(label = pct), position = position_dodge(width = .9), vjust = -.25, size = 2.5) +
  labs(title = "Myers Pitch Distribution Seen Vs RHP") + 
  theme(plot.title = element_text(face = "bold", hjust = .5)) -> wmyers19_R_pct_chart

#Find pitch distribution based on count vs RHP
round(prop.table(table(wmyers19_R$pitch_type, wmyers19_R$count),
                 margin = 2), 2) -> wmyers19_countsR
round(prop.table(table(wmyers20_R$pitch_type, wmyers20_R$count),
                 margin = 2), 2) -> wmyers20_countsR

#find first pitch swing % vs R
nrow(wmyers19_R %>% filter(description == "swinging_strike" | description == "hit_into_play" |
                      description == "foul" | description == "hit_into_play_no_out" |
                      description == "hit_into_play_score" | description == "foul_tip" |
                      description == "foul_bunt") %>% filter(balls == 0 & strikes == 0))/
  nrow(wmyers19_R %>% filter(balls == 0 & strikes == 0))

nrow(wmyers20_R %>% filter(description == "swinging_strike" | description == "hit_into_play" |
                           description == "foul" | description == "hit_into_play_no_out" |
                           description == "hit_into_play_score" | description == "foul_tip" |
                           description == "foul_bunt") %>% filter(balls == 0 & strikes == 0))/
  nrow(wmyers20_R %>% filter(balls == 0 & strikes == 0))

#create strike zone heat map function
k_zone_heatmap <- function(...) {
  k_zone_plot(...) +
    stat_density_2d(aes(fill = ..density..), geom = "raster", contour = FALSE) +
    scale_fill_distiller(palette= "Spectral", direction=-1) + 
    geom_rect(xmin = -(plate_width/2)/12,
              xmax = (plate_width/2)/12,
              ymin = 1.5,
              ymax = 3.6, color = "black", alpha = 0) +
    coord_equal() + 
    scale_x_continuous("Horizontal location (ft.)",
                       limits = c(-2,2)) +
    scale_y_continuous("Vertical location (ft.) RHB POV",
                       limits = c(0,5))
}

#Plot pitches seen with whiffs highlighted and classify swing and misses
wmyers19 %>% mutate(swing_and_miss = ifelse(wmyers19$description == "swinging_strike", "Whiff", "other")) -> wmyers19
wmyers20 %>% mutate(swing_and_miss = ifelse(wmyers20$description == "swinging_strike", "Whiff", "other")) -> wmyers20

wmyers19_R %>% 
  k_zone_heatmap(aes(plate_x, plate_z, color = swing_and_miss)) +
  geom_point(alpha = .5) + scale_color_manual(values = c("black", "red")) + 
  facet_wrap(~pitch_type, nrow = 3) +
  easy_add_legend_title("Swing and Miss?\n(yellow = Density)") + 
  labs(title = "Myers 2019 Pitches Seen vs RHP")

wmyers20_R %>% 
  k_zone_heatmap(aes(plate_x, plate_z, color = swing_and_miss)) +
  geom_point(alpha = .6) + scale_color_manual(values = c("black", "red")) + 
  facet_wrap(~pitch_type, nrow = 3) +
  easy_add_legend_title("Swing and Miss?\n(yellow = Density)") +
  labs(title = "Myers 2020 Pitches Seen vs RHP")
 
 #Create spray charts vs RHP separated by pitch type and colored by batted ball type
spray_chart(wmyers19_bip %>% filter(p_throws == "R"), aes(hc_x, -hc_y, color = bb_type)) +
  geom_point() + facet_wrap(~pitch_type, nrow = 3) + labs(title = "Myers 2019 Balls in Play vs RHP") +
  easy_add_legend_title("Batted Ball Type") -> wmyers19_R_pct
spray_chart(wmyers20_bip %>% filter(p_throws == "R"), aes(hc_x, -hc_y, color = bb_type)) +
  geom_point() + facet_wrap(~pitch_type, nrow = 3) + labs(title = "Myers 2020 Balls in Play vs RHP") +
  easy_add_legend_title("Batted Ball Type") -> wmyers20_R_pct
  
 
 ###################
 #VS lefties
  repeat all RHP code but replace with LHP
  

#Find % change in counts - Statistically insignificant so left out of write up
wmyers19 %>% 
  group_by(count) %>% 
  summarize(n = n(),
            pct19 = n/nrow(wmyers19)*100) %>% select(-n)
wmyers20 %>% 
  group_by(count) %>% 
  summarize(n = n(),
            pct20 = n/nrow(wmyers20)*100) %>% select(-n, -count)
cbind(wmyers19 %>% 
        group_by(count) %>% 
        summarize(n = n(),
                  pct19 = n/nrow(wmyers19)*100) %>% select(-n),wmyers20 %>% 
        group_by(count) %>% 
        summarize(n = n(),
                  pct20 = n/nrow(wmyers20)*100) %>% select(-n, -count)) %>% 
  mutate(pct_change = pct20 - pct19) -> wmyers_count_diff

(round(prop.table(table(wmyers20$count, wmyers20$year),
                 margin = 2), 2) - round(prop.table(table(wmyers19$count, wmyers19$year),
                                                    margin = 2), 2)) * 100

barplot(wmyers_count_diff$pct_change, names.arg = wmyers_count_diff$count)

#####################################

End
