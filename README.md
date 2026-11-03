# clic-project-figures
%% PLOTTING
% load results
dir = "C:\Users\franc\OneDrive\Desktop\barcellona\results - auditory discrimination task\";
addpath(dir)
data = 'data_03_20261103_091404';
trajectories = 'trajectories_03_20261103_091404';

results = readtable(data);
results(isnan(results.trial),:) = [];

MT = [results.MT];
idx = strcmp(MT, 'Too slow');
results(idx,:) = [];

trajectories = readtable(trajectories);
invalid = find(isnan(results.trial));
for i = 1:length(invalid)
    invalid_trial = invalid(i);
    trial_idx = trajectories.trial == invalid_trial;
    trajectories(trial_idx,:) = [];
end


%% RT against L-R
prob = [results.prob];
RT   = [results.RT];
pl   = cfg.pclickToEar;

for k = 1:numel(pl)
    idx  = prob == pl(k);
    m(k) = mean(RT(idx));
end

figure
plot(pl, m, 'o-', 'LineWidth', 1.5)
xlabel('L-R proportion')
xlim([min(pl) max(pl)])
ylabel('Mean reaction time (s)')

deltaLR = results.deltaLR;
plot(RT, deltaLR)
xlabel('L-R')
ylabel('Reaction time (s)')

%% movement time against L-R
deltaLR = [results.deltaLR];
MT   = [results.MT];
LR   = unique(deltaLR);

m = zeros(1,5);
idx  = deltaLR == LR(1);
m(1) = mean(MT(idx), 'omitnan');

idx  = deltaLR == LR(2);
m(2) = mean(MT(idx), 'omitnan');

idx  = deltaLR == LR(3);
m(3) = mean(MT(idx), 'omitnan');

idx  = deltaLR == LR(4);
m(4) = mean(MT(idx), 'omitnan');

idx  = deltaLR == LR(5);
m(5) = mean(MT(idx), 'omitnan');


figure
plot(LR, m, 'o-', 'LineWidth', 1.5)
xlabel('Proportion of clicks to the left ear')
ylabel('Mean movement time (s)')
xlim([min(LR)-0.05, max(LR)+0.05])

%% response likelihood against L-R
deltaLR = [results.deltaLR];
LR = unique(deltaLR);
accuracy = [results.accuracy_1];
acc = [];
for i = 1:length(LR)
    idx_acc = deltaLR == LR(i);
    acc = [acc mean(accuracy(idx_acc))];
end
plot(LR, acc)
xlabel('L-R')
ylabel('accuracy')
xlim([min(LR), max(LR)])

%% trajectories 
deltaLR = [results.deltaLR];
LR = unique(deltaLR);
allTraj = cell(1,length(LR));
allTrajLR = [];
nPoints = 1000;
tq = linspace(0, 1, nPoints);
for i = 1:length(LR)
    idx_trial = deltaLR == LR(i);
    allTrials = results.trial(idx_trial);
    allTrajLR = nan(nPoints, 2, length(allTrials));
    for k = 1:length(allTrials)
        idx = trajectories.trial == allTrials(k);
        tt = trajectories.time_s(idx);
        x  = trajectories.x_pix(idx);
        y  = trajectories.y_pix(idx);

        if numel(tt) < 2, continue, end               % no data for this trial

        [tt, u] = unique(tt);
        x = x(u);  y = y(u);
        if numel(tt) < 2, continue, end

        tn = (tt - tt(1)) / (tt(end) - tt(1));

        allTrajLR(:, 1, k) = interp1(tn, x, tq)';
        allTrajLR(:, 2, k) = interp1(tn, y, tq)';
    end

    allTraj{i} = mean(allTrajLR, 3, 'omitnan');
end

for i = 1:length(LR)
    m = allTraj{i};                    % 1000 x 2: column 1 = mean x, column 2 = mean y
    plot(m(:,1), m(:,2), 'DisplayName', sprintf('LR = %g', LR(i)));
    hold on
end
set(gca, 'YDir','reverse')
xlabel('x axis (px)')
ylabel('y axis (px)')
legend show
axis equal

%% performance improvement
results(isnan(results.trial),:) = [];
accuracy = results.accuracy_1;
trials = results.trial;

m100 = mean(accuracy(1:50));
std100 = std(accuracy(1:50));

m200 = mean(accuracy(51:100));
std200 = std(accuracy(51:100));

m300 = mean(accuracy(101:150));
std300 = std(accuracy(101:150));

m400 = mean(accuracy(151:end));
std400 = std(accuracy(151:end));

mean_all = [m100 m200 m300 m400];
std_all  = [std100 std200 std300 std400];
n_all    = [100 100 100 numel(accuracy(301:end))];
sem_all  = std_all ./ sqrt(n_all);

blockEnd = [50 100 150 200];   % x axis: number of trials completed

errorbar(blockEnd, mean_all, sem_all, '-o', 'LineWidth', 1.5)
xlabel('Number of trials')
ylabel('Accuracy')
xlim([0 blockEnd(end) + 50])


%% x and y position against time
x_position = trajectories.x_pix;
y_position = trajectories.y_pix;
time = trajectories.time_s;

plot(time, x_position)
xlabel('time (ms)')
ylabel('x position')

plot(time, y_position)
xlabel('time (ms)')
ylabel('y position')

plot(x_position, y_position)
xlabel('x position')
ylabel('y position')
set(gca,'YDir','reverse')
